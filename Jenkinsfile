pipeline {
    agent any

    environment {
        GIT_REPO = 'git@github.com:MdHakkim/real_pipeline.git'
        BRANCH = "${env.BRANCH_NAME}"
        PROJECT = "laravel_${env.BRANCH_NAME.replaceAll('/', '_')}"
        APP_NAME = "${PROJECT}_app_1"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Cleanup Old Containers') {
            steps {
                sh """
                docker-compose -p ${PROJECT} down --remove-orphans --volumes || true
                """
            }
        }

        stage('Copy .env') {
            steps {
                sh """
                rm -f .env
                cp /var/www/project/.env .env
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker-compose -p ${PROJECT} build"
            }
        }

        stage('Run Containers') {
            steps {
                sh "docker-compose -p ${PROJECT} up -d"
            }
        }

        stage('Wait') {
            steps {
                sh "sleep 10"
            }
        }

        stage('Laravel Commands') {
            steps {
                sh """
                docker exec ${APP_NAME} composer install --no-dev --optimize-autoloader
                docker exec ${APP_NAME} php artisan migrate --force
                docker exec ${APP_NAME} php artisan config:cache
                docker exec ${APP_NAME} php artisan route:cache
                docker exec ${APP_NAME} php artisan view:cache
                """
            }
        }

        stage('Tests') {
            steps {
                sh "docker exec ${APP_NAME} ./vendor/bin/phpunit || true"
            }
        }

        stage('Health Check') {
            steps {
                sh "curl -f http://192.168.21.128:8082/ || exit 1"
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline ${PROJECT} succeeded!"
        }
        failure {
            echo "❌ Pipeline ${PROJECT} failed!"
        }
    }
}
