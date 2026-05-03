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

        stage('📥 Pull Latest Image') {
            steps {
                sh '''
                    docker pull $IMAGE_NAME:latest
                '''
            }
        }

        stage('🚀 Deploy on Server') {
            steps {
                sshagent(['deployment-server-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -p $DEPLOY_PORT $DEPLOY_USER@$DEPLOY_SERVER "
                            set -e

                            echo '🚀 Deploying Bookstore App...'

                            # Pull latest image
                            docker pull $IMAGE_NAME:latest

                            # Stop old container if exists
                            docker stop $APP_NAME || true
                            docker rm $APP_NAME || true

                            # Run new container
                            docker run -d \
                                --name $APP_NAME \
                                --restart unless-stopped \
                                --env-file $ENV_FILE \
                                -p $APP_PORT:8000 \
                                $IMAGE_NAME:latest

                            sleep 5

                            echo '📦 Running Containers:'
                            docker ps | grep $APP_NAME || true

                            echo '📜 Last Logs:'
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
                        echo "Waiting for app..."
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