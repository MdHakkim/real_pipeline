pipeline {
    agent any

    environment {
        GIT_REPO = 'git@github.com:MdHakkim/real_pipeline.git'
        BRANCH = 'main'
        APP_NAME = 'laravel_app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker-compose build"
            }
        }

        stage('Run Containers') {
            steps {
                sh "docker-compose up -d"
            }
        }

        stage('Laravel Commands') {
            steps {
                sh "docker exec ${APP_NAME} php artisan migrate --force"
                sh "docker exec ${APP_NAME} php artisan config:cache"
                sh "docker exec ${APP_NAME} php artisan route:cache"
                sh "docker exec ${APP_NAME} php artisan view:cache"
            }
        }

        stage('Tests') {
            steps {
                sh "docker exec ${APP_NAME} ./vendor/bin/phpunit"
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded, Laravel app is running in Docker!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
