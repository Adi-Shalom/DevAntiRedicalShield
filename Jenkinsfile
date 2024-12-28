pipeline {
    agent any

    stages {
        stage('Checkout Dev Configurations') {
            steps {
                dir('DevAntiRedicalShield') { // יצירת תיקייה עבור DevAntiRedicalShield
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[url: 'https://github.com/Adi-Shalom/DevAntiRedicalShield.git']]
                    ])
                }
            }
        }

        stage('Checkout Application Code') {
            steps {
                dir('AntiRedicalShield') { // יצירת תיקייה עבור AntiRedicalShield
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
                dir('AntiRedicalShield') { // בדיקה תחבירית בתוך תיקיית AntiRedicalShield
                    sh 'python3 -m py_compile app.py'
                }
            }
        }

        stage('Run Application and Test') {
            steps {
                dir('AntiRedicalShield') { // הרצת היישום ובדיקות בתיקיית AntiRedicalShield
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
                dir('AntiRedicalShield') { // בניית תמונת Docker מתוך תיקיית AntiRedicalShield
                    sh 'docker build -t adishalom/antiredicalshield:latest .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    dir('AntiRedicalShield') { // דחיפת תמונת Docker מתוך AntiRedicalShield
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
                        echo "Deploying manifests to namespace antiradicalshield..."
                        if [ -d DevAntiRedicalShield ]; then
                            kubectl apply -f DevAntiRedicalShield/ --validate=false
                        else
                            kubectl apply -f . --namespace=antiradicalshield --validate=false
                        fi
                        '''
                    }
                }
            }
        }
    }
}
