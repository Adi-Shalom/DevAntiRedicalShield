pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Syntax Check') {
            steps {
                sh 'python -m py_compile app.py'
            }
        }

        stage('Run Application and Test') {
            steps {
                script {
                    sh '''
                    python app.py &
                    sleep 5
                    curl -I http://localhost:5000
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t adishalom/antiredicalshield:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    sh 'docker push adishalom/antiredicalshield:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
            }
        }
    }
}
