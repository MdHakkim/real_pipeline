pipeline {
    agent any

    environment {
        GIT_REPO = 'git@github.com:MdHakkim/real_pipeline.git'
        BRANCH = "${env.BRANCH_NAME}"
    }

    stages {

        stage('Init') {
            steps {
                script {

                    // SAFE project naming per branch
                    env.PROJECT = "laravel_${env.BRANCH_NAME}".replaceAll("/", "_")

                    // Unique containers per branch
                    env.APP_NAME = "${env.PROJECT}_app"
                    env.DB_NAME = "${env.PROJECT}_db"

                    // Unique ports per branch (avoid conflict)
                    def basePort = env.BRANCH_NAME == "main" ? 8080 :
                                   env.BRANCH_NAME == "dev" ? 8090 : 8100

                    env.WEB_PORT = (basePort + env.BUILD_NUMBER.toInteger()).toString()
                    env.DB_PORT = (3300 + env.BUILD_NUMBER.toInteger()).toString()

                    echo "PROJECT = ${env.PROJECT}"
                    echo "APP_NAME = ${env.APP_NAME}"
                    echo "WEB_PORT = ${env.WEB_PORT}"
                }
            }
        }

        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Cleanup OLD BRANCH STACK ONLY') {
            steps {
                sh '''
                    echo "Cleaning only this branch stack..."

                    docker-compose -p $PROJECT down -v --remove-orphans || true

                    docker rm -f $(docker ps -aq --filter "name=$PROJECT") || true
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

        stage('Run') {
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
                    curl -f http://localhost:$WEB_PORT || exit 1
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment SUCCESS for branch: $BRANCH"
        }
        failure {
            echo "❌ Deployment FAILED for branch: $BRANCH"
        }
    }
}
