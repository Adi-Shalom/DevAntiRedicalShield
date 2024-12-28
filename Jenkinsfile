pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'adishalom/antiredicalshield:latest'
        K8S_NAMESPACE = 'antiradicalshield'
        GITHUB_DEV_REPO = 'https://github.com/Adi-Shalom/DevAntiRedicalShield.git'
        GITHUB_APP_REPO = 'https://github.com/Adi-Shalom/AntiRedicalShield.git'
    }

    stages {
        stage('Pre-checks') {
            steps {
                script {
                    echo 'Checking if Minikube is running...'
                    def minikubeStatus = sh(script: 'minikube status', returnStatus: true)

                    if (minikubeStatus != 0) {
                        echo 'Minikube is not running. Starting Minikube...'
                        sh 'minikube start --driver=docker'
                    } else {
                        echo 'Minikube is already running.'
                    }

                    echo 'Verifying Kubernetes cluster...'
                    sh 'kubectl cluster-info'
                }
            }
        }

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
                        '''
                    }
                }
            }
        }

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

