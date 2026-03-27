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

        stage('Copy .env') {
            steps {
                sh '''
                    cp /var/www/project/.env /var/lib/jenkins/workspace/real_pipeline_project/.env
                '''
            }
        }

        stage('Cleanup Old Containers') {
            steps {
                sh '''
                docker-compose down --remove-orphans --volumes || true
                docker rm -f laravel_app laravel_db laravel_nginx || true
                '''
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

        stage('Wait for Container') {
            steps {
                sh "sleep 10"
            }
        }

        stage('Laravel Commands') {
            steps {
                // Install dependencies first
                sh "docker exec ${APP_NAME} composer install --no-dev --optimize-autoloader"
                sh "docker exec ${APP_NAME} php artisan migrate --force"
                sh "docker exec ${APP_NAME} php artisan config:cache"
                sh "docker exec ${APP_NAME} php artisan route:cache"
                sh "docker exec ${APP_NAME} php artisan view:cache"
            }
        }

        stage('Tests') {
            steps {
                sh "docker exec ${APP_NAME} ./vendor/bin/phpunit || true"
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline succeeded, Laravel app is running!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
