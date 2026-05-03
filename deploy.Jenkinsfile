pipeline {
    agent any

    environment {
        APP_NAME      = "bookstore-django"
        IMAGE_NAME    = "prabeshbuilds/bookstore-django"
        DEPLOY_SERVER = "52.87.213.119"
        DEPLOY_USER   = "ubuntu"
        DEPLOY_PORT   = "22"

        APP_PORT      = "8000"
        ENV_FILE      = "/home/ubuntu/.env"
    }

    stages {

        stage('🔨 Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME:latest .
                '''
            }
        }

        stage('🔨 Build Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                    set -e
                    echo "🔨 Building Docker image..."
                    docker build --pull -t $DOCKER_USERNAME/$APP_NAME:$IMAGE_TAG .
                    docker tag $DOCKER_USERNAME/$APP_NAME:$IMAGE_TAG $DOCKER_USERNAME/$APP_NAME:latest
                    '''
                }
            }
        }

        stage('📤 Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                    set -e
                    echo "📤 Logging in to DockerHub..."
                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                    echo "📤 Pushing Docker images..."
                    docker push $DOCKER_USERNAME/$APP_NAME:$IMAGE_TAG
                    docker push $DOCKER_USERNAME/$APP_NAME:latest
                    '''
                }
            }
        }

        stage('🚀 Deploy on Server') {
            steps {
                sshagent(['deployment-server-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -p $DEPLOY_PORT $DEPLOY_USER@$DEPLOY_SERVER "
                            set -e

                            echo '🚀 Deploying Bookstore App...'

                            docker pull $IMAGE_NAME:latest

                            docker stop $APP_NAME || true
                            docker rm $APP_NAME || true

                            docker run -d \
                                --name $APP_NAME \
                                --restart unless-stopped \
                                --env-file $ENV_FILE \
                                -p $APP_PORT:8000 \
                                $IMAGE_NAME:latest

                            sleep 5

                            docker ps | grep $APP_NAME || true
                            docker logs --tail 20 $APP_NAME || true
                        "
                    '''
                }
            }
        }

        stage('🔍 Health Check') {
            steps {
                sh '''
                    echo "Checking application..."

                    for i in $(seq 1 10); do
                        if curl -s http://$DEPLOY_SERVER:$APP_PORT | grep -q "Django"; then
                            echo "✅ App is running successfully"
                            exit 0
                        fi
                        sleep 5
                    done

                    echo "❌ Health check failed"
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful"
        }
        failure {
            echo "❌ Deployment Failed"
        }
    }
}