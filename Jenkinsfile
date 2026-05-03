pipeline {
    agent any

    environment {
        IMAGE_NAME = "prabeshbuilds/bookstore-django"
        DB_NAME = "bookstore"
        DB_USER = "postgres"
        DB_PASSWORD = "postgres"
        DB_HOST = "localhost"
        DB_PORT = "5432"
    }

    stages {

        stage('📥 Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/prabeshbuilds/todo.git'
            }
        }

        stage('🐍 Setup Environment') {
             steps {
            sh '''
                python3 -m venv env
                env/bin/python -m pip install --upgrade pip
                env/bin/python -m pip install -r requirements.txt
                env/bin/python -m pip install flake8
            '''
            }
        }

        // stage('🗄️ Run Migrations') {
        //     steps {
        //         sh '''
                
        //             python3 manage.py migrate
        //         '''
        //     }
        // }

        // stage('🧹 Lint Code') {
        //     steps {
        //         sh '''
        //             python3 -m venv env
        //             flake8 .
        //         '''
        //     }
        // }

        stage('🧪 Run Tests') {
            steps {
                sh '''
                    source env/bin/activate
                    python3 manage.py test
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

        // OPTIONAL: Uncomment if you want Docker Hub push
        /*
        stage('📤 Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }
        */

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