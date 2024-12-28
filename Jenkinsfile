pipeline {
    agent any

    stages {
        stage('Checkout Dev Configurations') {
            steps {
                dir('DevAntiRedicalShield') {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[url: 'https://github.com/Adi-Shalom/DevAntiRadicalShield.git']]
                    ])
                } // כאן נסגר הבלוק של dir בצורה נכונה
            }
        }

        stage('Checkout Application Code') {
            steps {
                dir('AntiRedicalShield') {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[url: 'https://github.com/Adi-Shalom/AntiRedicalShield.git']]
                    ])
                }
            }
        }

        stage('Syntax Check') {
            steps {
                dir('AntiRedicalShield') {
                    sh 'python3 -m py_compile app.py'
                }
            }
        }

        stage('Run Application and Test') {
            steps {
                dir('AntiRedicalShield') {
                    script {
                        sh '''
                        python3 app.py &
                        sleep 5
                        curl -I http://localhost:5000
                        pkill -f app.py
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('AntiRedicalShield') {
                    sh 'docker build -t adishalom/antiredicalshield:latest .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    dir('AntiRedicalShield') {
                        sh 'docker push adishalom/antiredicalshield:latest'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                    script {
                        sh '''
                        echo "Checking Kubernetes connection..."
                        kubectl cluster-info

                        echo "Ensuring namespace 'antiradicalshield' exists..."
                        kubectl get namespace antiradicalshield || kubectl create namespace antiradicalshield

                        echo "Deploying manifests..."
                        if [ -d DevAntiRedicalShield ]; then
                            kubectl apply -f DevAntiRedicalShield/ --namespace=antiradicalshield --validate=false
                        else
                            echo "Error: DevAntiRedicalShield directory not found"
                            exit 1
                        fi
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            sh 'docker image prune -f'
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for errors.'
        }
    }
}
