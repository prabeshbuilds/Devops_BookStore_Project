pipeline {
    agent any

    environment {
        IMAGE_NAME = "prabeshbuilds/bookstore-django"
    }

    stages {

        stage('📥 Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/prabeshbuilds/Devops_BookStore_Project.git'
            }
        }

        stage('🐍 Setup Virtual Environment') {
            steps {
                sh '''
                    python3 -m venv env

                    # ALWAYS use venv python (IMPORTANT FIX)
                    env/bin/python -m pip install --upgrade pip
                    env/bin/python -m pip install -r requirements.txt
                '''
            }
        }

        stage('🗄️ Run Migrations') {
            steps {
                sh '''
                    env/bin/python manage.py migrate
                '''
            }
        }

        stage('🧪 Run Tests') {
            steps {
                sh '''
                    env/bin/python manage.py test
                '''
            }
        }

        stage('🧹 Lint Code') {
            steps {
                sh '''
                    env/bin/python -m pip install flake8
                    env/bin/python -m flake8 .
                '''
            }
        }

        stage('🐳 Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('📤 Optional Docker Run Test') {
            steps {
                sh '''
                    docker run --rm ${IMAGE_NAME}:latest echo "Container works"
                '''
            }
        }
    }

    post {
        success {
            echo '✅ CI Pipeline Completed Successfully'
        }
        failure {
            echo '❌ CI Pipeline Failed'
        }
        always {
            cleanWs()
        }
    }
}