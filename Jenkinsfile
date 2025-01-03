pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'adishalom/antiredicalshield:latest'
        K8S_NAMESPACE = 'antiradicalshield'
        GITHUB_DEV_REPO = 'https://github.com/Adi-Shalom/DevAntiRedicalShield.git'
        GITHUB_APP_REPO = 'https://github.com/Adi-Shalom/AntiRedicalShield.git'
    }

    stages {
        stage('Checkout Repositories') {
            parallel {
                stage('Checkout Dev Configurations') {
                    steps {
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: '*/main']],
                            userRemoteConfigs: [[url: "${GITHUB_DEV_REPO}"]]
                        ])
                    }
                }

                stage('Checkout Application Code') {
                    steps {
                        dir('AntiRedicalShield') {
                            checkout([
                                $class: 'GitSCM',
                                branches: [[name: '*/main']],
                                userRemoteConfigs: [[url: "${GITHUB_APP_REPO}"]]
                            ])
                        }
                    }
                }
            }
        }

        stage('Prepare Application') {
            parallel {
                stage('Install Dependencies') {
                    steps {
                        dir('AntiRedicalShield') {
                            sh '''
                            echo "Installing Python dependencies..."
                            pip3 install -r requirements.txt
                            '''
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
            }
        }

        stage('Run and Test Application') {
            steps {
                dir('AntiRedicalShield') {
                    script {
                        sh '''
                        echo "Starting application for testing..."
                        python3 app.py &
                        sleep 5
                        echo "Testing application endpoint..."
                        curl -I http://localhost:5000
                        '''
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            parallel {
                stage('Build Docker Image') {
                    steps {
                        dir('AntiRedicalShield') {
                            sh "docker build -t ${DOCKER_IMAGE} ."
                        }
                    }
                }

                stage('Push Docker Image') {
                    steps {
                        withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                            dir('AntiRedicalShield') {
                                sh "docker push ${DOCKER_IMAGE}"
                            }
                        }
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

                        echo "Ensuring namespace '${K8S_NAMESPACE}' exists..."
                        kubectl get namespace ${K8S_NAMESPACE} || kubectl create namespace ${K8S_NAMESPACE}

                        echo "Deploying manifests..."
                        if [ -d DevAntiRedicalShield ]; then
                            kubectl apply -f DevAntiRedicalShield/ --namespace=${K8S_NAMESPACE} --validate=false
                        else
                            kubectl apply -f . --namespace=${K8S_NAMESPACE} --validate=false
                        fi
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up Docker images..."
            sh 'docker image prune -f'

            echo "Final pipeline status: ${currentBuild.currentResult}"
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for errors.'
        }
    }
}
