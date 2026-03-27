pipeline {
    agent any

    environment {
        GIT_REPO = 'git@github.com:MdHakkim/real_pipeline.git'
        BRANCH = 'main'
        PROJECT = "laravel_${env.BRANCH_NAME ?: 'main'}"
    }

    stages {
        stage('Init Ports') {
            steps {
                script {
                    env.DB_PORT = (3300 + env.BUILD_NUMBER.toInteger()).toString()
                    env.WEB_PORT = (8080 + env.BUILD_NUMBER.toInteger()).toString()
                    env.PROJECT = "laravel_${env.BRANCH_NAME ?: 'main'}"
                    env.APP_NAME = "${env.PROJECT}_app_1"
        
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

        stage('Copy .env') {
            steps {
                sh '''
                set -e
    
                echo "Cleaning old .env if exists..."
                rm -rf /var/lib/jenkins/workspace/real_pipeline_project/.env
    
                echo "Copying .env..."
                cp /var/www/project/.env /var/lib/jenkins/workspace/real_pipeline_project/.env
    
                ls -l /var/lib/jenkins/workspace/real_pipeline_project/.env
            '''
            }
        }

        stage('Cleanup Old Containers') {
            steps {
                sh '''
                docker-compose -p ${PROJECT} down --remove-orphans --volumes || true
                docker system prune -f || true
                '''
            }
        }
        stage('Debug Info') {
            steps {
                sh '''
                echo "BRANCH = $BRANCH"
                echo "PROJECT = $PROJECT"
                docker ps -a
                docker network ls
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
                sh 'docker exec ${APP_NAME} ./vendor/bin/phpunit || true'
            }
        }
        
        stage('Run Migrations') {
            steps {
                sh '''
                    docker exec ${APP_NAME} php artisan migrate --force
                '''
            }
        }
        stage('Laravel Optimize') {
            steps {
                sh '''
                    docker exec ${APP_NAME} php artisan config:clear
                    docker exec ${APP_NAME} php artisan cache:clear
                    docker exec ${APP_NAME} php artisan route:clear
                    docker exec ${APP_NAME} php artisan config:cache
                '''
            }
        }
        stage('Health Check') {
            steps {
                sh '''
                    curl -f http://192.168.21.128:8082/ || exit 1
                '''
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
