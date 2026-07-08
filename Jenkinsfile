pipeline {
    agent any

    // skipDefaultCheckout: prevents Jenkins from silently cloning the repo
    // before the pipeline runs. Our explicit 'Checkout' stage below handles
    // the clone, so it appears as a named, visible stage in the Stage View.
    options {
        skipDefaultCheckout(true)
    }

    environment {
        IMAGE_NAME      = 'juice-shop'
        CONTAINER_NAME  = 'juice-shop-dev'
        APP_PORT        = '3000'
        // NVD API key stored as a Jenkins Secret Text credential.
        // Never hardcoded — Jenkins injects and masks it at runtime.
        NVD_API_KEY     = credentials('NVD_API_KEY')
    }

    stages {

        // ─────────────────────────────────────────────────────────────────
        // STAGE 1 — Checkout
        // Explicitly clones the repo from SCM into the Jenkins workspace.
        // ─────────────────────────────────────────────────────────────────
        stage('Checkout') {
            steps {
                checkout scm
                echo "✅ Checkout complete — workspace: ${env.WORKSPACE}"
            }
        }

        // ─────────────────────────────────────────────────────────────────
        // STAGE 2 — Install Dependencies
        // Installs all project dependencies.
        // The project's npm lifecycle scripts perform any additional
        // build steps required by the application.
        // ─────────────────────────────────────────────────────────────────
        stage('Install Dependencies') {
            steps {
                bat 'node --version'
                bat 'npm --version'
                bat 'npm install'
                echo '✅ Dependencies installed and build complete.'
            }
        }

        // ─────────────────────────────────────────────────────────────────
        // STAGE 3 — OWASP Dependency Check
        // Scans project dependencies against the NVD (National Vulnerability
        // Database) to identify known CVEs.
        // --failOnCVSS 11 : CVSS scores are 0-10, so this never blocks the
        //   build. For this assessment the goal is detection and reporting,
        //   not build gating.
        // --nvdApiKey    : Authenticates with NVD API to avoid rate-limiting
        //   on the initial database download. Key is injected from a Jenkins
        //   Secret Text credential — never stored in source control.
        // Reports are written to the 'reports/' directory.
        // Publishing is handled in the pipeline-level post/always block so
        // the report surfaces even if a later stage (Docker Build, Deploy)
        // fails.
        // ─────────────────────────────────────────────────────────────────
        // stage('OWASP Dependency Check') {
        //     steps {
        //         bat 'if not exist reports mkdir reports'
        //         dependencyCheck additionalArguments: """
        //             --scan ./
        //             --format HTML
        //             --format XML
        //             --out reports/
        //             --prettyPrint
        //             --failOnCVSS 11
        //             --nvdApiKey ${NVD_API_KEY}
        //         """, odcInstallation: 'OWASP-DC'
        //     }
        // }

        // ─────────────────────────────────────────────────────────────────
        // STAGE 4 — Docker Build
        // Builds the Docker image using the existing project Dockerfile.
        // Tagged with BUILD_NUMBER for traceability across builds.
        // ─────────────────────────────────────────────────────────────────
        stage('Docker Build') {
            steps {
                bat "docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} ."
                echo "✅ Docker image built: ${IMAGE_NAME}:${env.BUILD_NUMBER}"
            }
        }

        // ─────────────────────────────────────────────────────────────────
        // STAGE 5 — Stop & Remove Old Container
        // Cleans up any previously running container before deploying.
        // '|| echo' prevents the stage from failing if no container exists,
        // ensuring the pipeline is idempotent across repeated runs.
        // ─────────────────────────────────────────────────────────────────
        stage('Stop & Remove Old Container') {
            steps {
                script {
                    bat(returnStatus: true, script: "docker stop ${CONTAINER_NAME}")
                    bat(returnStatus: true, script: "docker rm ${CONTAINER_NAME}")
                }

                echo '✅ Old container cleared.'
            }
        }

        // ─────────────────────────────────────────────────────────────────
        // STAGE 6 — Deploy Container
        // Runs the newly built image as a detached container on APP_PORT.
        // ─────────────────────────────────────────────────────────────────
        stage('Deploy Container') {
            steps {
                bat "docker run -d -p ${APP_PORT}:3000 --name ${CONTAINER_NAME} ${IMAGE_NAME}:${env.BUILD_NUMBER}"
                echo "✅ Container '${CONTAINER_NAME}' started on port ${APP_PORT}."
            }
        }

        // ─────────────────────────────────────────────────────────────────
        // STAGE 7 — Health Check
        // Polls the application until it responds or retries are exhausted.
        // Retries up to 10 times with a 10-second pause between attempts,
        // giving the app up to 100 seconds to become available.
        // ─────────────────────────────────────────────────────────────────
        stage('Health Check') {
            steps {
                script {
                    retry(10) {
                        sleep(time: 10, unit: 'SECONDS')
                        bat "curl -f http://localhost:${APP_PORT}"
                    }
                }
                echo "✅ Application is live at http://localhost:${APP_PORT}"
            }
        }

    } // end stages

    post {
        always {
            echo 'Dependency Check temporarily disabled.'
            // Publish the DC report regardless of which stage failed.
            // The report is the primary deliverable of this pipeline.
            // dependencyCheckPublisher pattern: 'reports/dependency-check-report.xml'
        }
        success { echo '🟢 Pipeline succeeded.' }
        failure { echo '🔴 Pipeline failed.' }
    }
}
