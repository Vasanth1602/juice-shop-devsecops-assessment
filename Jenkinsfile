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

    } // end stages

    post {
        success { echo '🟢 Pipeline succeeded.' }
        failure { echo '🔴 Pipeline failed.' }
    }
}
