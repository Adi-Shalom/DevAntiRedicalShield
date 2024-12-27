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
                    sh '''
                    # הוספת הנתיב עבור חבילות מקומיות
                    export PATH=$PATH:$HOME/.local/bin
                    # התקנת כל התלויות
                    pip install --user -r requirements.txt
                    # הרצת בדיקות
                    pytest
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    sh '''
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    git url: 'https://github.com/Adi-Shalom/DevAntiRedicalShield.git', branch: 'main'
                    sh '''
                    kubectl apply -f flask-deployment.yml
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
