pipeline {
    agent any

    triggers {
    githubPush()
    }

    environment {
        APP_NAME      = "bookstore-django"
        IMAGE_TAG     = "${env.GIT_COMMIT.take(7)}"
        IMAGE_NAME = "prabeshdevops/bookstore-django"

        DEPLOY_SERVER = "13.221.202.181"
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

                        docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('🚀 Deploy on Server') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sshagent(['deployment-ssh']) {
                        sh '''
                            set -e

                            ssh -o StrictHostKeyChecking=no -p $DEPLOY_PORT $DEPLOY_USER@$DEPLOY_SERVER "
                                set -e

                                echo '🚀 Deploying Bookstore App...'

                                echo \"$DOCKER_PASSWORD\" | docker login -u \"$DOCKER_USERNAME\" --password-stdin

                                docker pull $IMAGE_NAME:$IMAGE_TAG

                                docker stop $APP_NAME || true
                                docker rm $APP_NAME || true

                                docker run -d \
                                    --name $APP_NAME \
                                    --restart unless-stopped \
                                    --env-file $ENV_FILE \
                                    -p $APP_PORT:8000 \
                                    $IMAGE_NAME:$IMAGE_TAG

                                sleep 5

                                docker ps | grep $APP_NAME || true
                                docker logs --tail 20 $APP_NAME || true
                            "
                        '''
                    }
                }
            }
        }

        stage('🔍 Health Check') {
            steps {
                    sh '''
                        echo "Checking application..."

                        echo "Server: $DEPLOY_SERVER:8034"

                        curl -v --max-time 5 http://$DEPLOY_SERVER:8034 || echo "❌ Curl failed"

                        echo "Done"
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