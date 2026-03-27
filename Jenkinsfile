pipeline {
    agent any

    environment {
        GIT_REPO = 'git@github.com:MdHakkim/real_pipeline.git'
        BRANCH = 'main'
    }

    stages {

        stage('Init') {
            steps {
                script {
                    env.PROJECT = "laravel_${env.BRANCH_NAME ?: 'main'}"
                    env.APP_NAME = "${env.PROJECT}_app_1"
                    env.DB_NAME = "${env.PROJECT}_db_1"
                    env.NETWORK = "${env.PROJECT}_default"

                    echo "PROJECT = ${env.PROJECT}"
                    echo "APP_NAME = ${env.APP_NAME}"
                }
            }
        }

        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Cleanup EVERYTHING') {
            steps {
                sh '''
                    echo "Stopping old stack..."
                    docker-compose -p $PROJECT down -v --remove-orphans || true

                    echo "Removing containers by name..."
                    docker rm -f $(docker ps -aq --filter "name=${PROJECT}") || true

                    echo "Removing old images (safe cleanup)..."
                    docker image prune -af || true

                    echo "Cleaning unused networks..."
                    docker network prune -f || true
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker-compose -p $PROJECT build --no-cache
                '''
            }
        }

        stage('Run Containers') {
            steps {
                sh '''
                    docker-compose -p $PROJECT up -d
                '''
            }
        }

        stage('Wait') {
            steps {
                sh 'sleep 15'
            }
        }

        stage('Laravel Setup') {
            steps {
                sh '''
                    docker exec $APP_NAME composer install --no-interaction --no-dev --optimize-autoloader
                    docker exec $APP_NAME php artisan migrate --force
                    docker exec $APP_NAME php artisan config:cache
                    docker exec $APP_NAME php artisan route:cache
                    docker exec $APP_NAME php artisan view:cache
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    curl -f http://localhost:8082 || exit 1
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful for $PROJECT"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
