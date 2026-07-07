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
        // Reports are written to the 'reports/' directory.
        // Publishing is handled in the pipeline-level post/always block so
        // the report surfaces even if a later stage (Docker Build, Deploy)
        // fails.
        // NOTE: First run downloads ~500 MB of NVD data. Expect 15-30 min.
        // ─────────────────────────────────────────────────────────────────
        stage('OWASP Dependency Check') {
            steps {
                bat 'if not exist reports mkdir reports'
                dependencyCheck additionalArguments: '''
                    --scan ./
                    --format HTML
                    --format XML
                    --out reports/
                    --prettyPrint
                    --failOnCVSS 11
                ''', odcInstallation: 'OWASP-DC'
            }
        }

    } // end stages

    post {
        always {
            // Publish the DC report regardless of which stage failed.
            // The report is the primary deliverable of this pipeline.
            dependencyCheckPublisher pattern: 'reports/dependency-check-report.xml'
        }
        success { echo '🟢 Pipeline succeeded.' }
        failure { echo '🔴 Pipeline failed.' }
    }
}
