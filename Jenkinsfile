pipeline {
    agent any

    stages {
        stage('Checkout Dev Configurations') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/Adi-Shalom/DevAntiRedicalShield.git']]
                ])
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
                script {
                    sh '''
                    # בדוק אם הקבצים קיימים בתיקייה DevAntiRedicalShield
                    if [ -d DevAntiRedicalShield ]; then
                        echo "Applying manifests from DevAntiRedicalShield directory..."
                        kubectl apply -f DevAntiRedicalShield/ --validate=false
                    else
                        echo "Applying manifests from the current directory..."
                        kubectl apply -f . --validate=false
                    fi
                    '''
                }
            }
        }
    }
}
