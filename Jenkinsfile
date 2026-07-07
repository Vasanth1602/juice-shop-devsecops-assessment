pipeline {
    agent any

    environment {
        IMAGE_NAME      = 'juice-shop'
        CONTAINER_NAME  = 'juice-shop-dev'
        APP_PORT        = '3000'
    }

    stages {

        // ─────────────────────────────────────────────────────────────────
        // STAGE 1 — Checkout
        // Pulls the source code from SCM (Git).
        // ─────────────────────────────────────────────────────────────────
        stage('Checkout') {
            steps {
                checkout scm
                echo "✅ Checkout complete — workspace: ${env.WORKSPACE}"
            }
        }

    } // end stages

    post {
        success { echo '🟢 Pipeline succeeded.' }
        failure { echo '🔴 Pipeline failed.' }
    }
}
