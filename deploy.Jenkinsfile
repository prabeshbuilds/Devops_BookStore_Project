pipeline {
    agent any

    environment {
        APP_NAME      = "bookstore-django"
        IMAGE_NAME    = "prabeshbuilds/bookstore-django"
        IMAGE_TAG     = "latest"

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
                    set -e
                    echo "🔨 Building Docker image..."
                    docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
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

                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin


                        docker push $DOCKER_USERNAME/$APP_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('🚀 Deploy on Server') {
            steps {
                sshagent(['deployment-server-ssh']) {
                    sh '''
                        set -e

                        ssh -o StrictHostKeyChecking=no -p $DEPLOY_PORT $DEPLOY_USER@$DEPLOY_SERVER "
                            set -e

                            echo '🚀 Deploying Bookstore App...'

                            docker pull prabeshbuilds/$APP_NAME:$IMAGE_TAG

                            docker stop $APP_NAME || true
                            docker rm $APP_NAME || true

                            docker run -d \
                                --name $APP_NAME \
                                --restart unless-stopped \
                                --env-file $ENV_FILE \
                                -p $APP_PORT:8000 \
                                prabeshbuilds/$APP_NAME:$IMAGE_TAG

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
                        if curl -s --max-time 5 http://$DEPLOY_SERVER:$APP_PORT | grep -i "django"; then
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