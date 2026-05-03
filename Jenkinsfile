pipeline {
    agent any

    environment {
        IMAGE_NAME = "prabeshbuilds/bookstore-django"
    }

    stages {

        stage('📥 Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/prabeshbuilds/Devops_BookStore_Project.git'
            }
        }

        stage('🐳 Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:latest .
                '''
            }
        }

    //    stage('🧪 Run Container Test') {
    //         steps {
    //         sh '''
    //         docker run --rm -w /app \
    //         prabeshbuilds/bookstore-django:latest \
    //         python manage.py check
    //       '''
    //         }
    //     }
        stage('📤 Push Image (Optional)') {
            steps {
                echo "Add DockerHub push here if needed"
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD SUCCESS"
        }
        failure {
            echo "❌ CI FAILED"
        }
        always {
            cleanWs()
        }
    }
}