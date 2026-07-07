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
        // Runs 'npm install' at the repo root.
        // The postinstall hook automatically:
        //   1. Installs frontend deps (cd frontend && npm install)
        //   2. Builds the Angular frontend (npm run build:frontend)
        //   3. Compiles TypeScript backend (npm run build:server)
        // This produces node_modules/ ready for OWASP DC to scan.
        // ─────────────────────────────────────────────────────────────────
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
                echo '✅ Dependencies installed and build complete.'
            }
        }

    } // end stages

    post {
        success { echo '🟢 Pipeline succeeded.' }
        failure { echo '🔴 Pipeline failed.' }
    }
}
