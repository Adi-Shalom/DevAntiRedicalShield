pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'adishalom/antiredicalshield:latest'
    }

    stages {
        stage('Checkout Backend and Frontend Code') {
            steps {
                git url: 'https://github.com/Adi-Shalom/AntiRedicalShield.git', branch: 'main'
            }
        }

        stage('Test and Build') {
            steps {
                script {
                    sh 'pip install -r requirements.txt'
                    sh 'pytest' // התאימי במידת הצורך
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                git url: 'https://github.com/Adi-Shalom/DevAntiRedicalShield.git', branch: 'main'
                sh 'kubectl apply -f deployment.yaml'
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed successfully.'
        }
    }
}

