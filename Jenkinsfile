pipeline {
    agent any

    stages {
        stage('Checkout Dev Configurations') {
            steps {
                dir('DevAntiRadicalShield') { // יצירת תיקייה עבור DevAntiRadicalShield
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[url: 'https://github.com/Adi-Shalom/DevAntiRadicalShield.git']]
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
                        pkill -f "python3 app.py" || true
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
                        kubectl --kubeconfig=DevAntiRedicalShield/kubeconfig cluster-info
                        echo "Deploying manifests to namespace antiradicalshield..."
                        kubectl --kubeconfig=DevAntiRadicalShield/kubeconfig apply -f DevAntiRedicalShield/ --validate=false
                        '''
                    }
                }
            }
        }
    }
}
