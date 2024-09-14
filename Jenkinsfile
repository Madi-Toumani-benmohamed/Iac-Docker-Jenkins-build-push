pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'dev', url: 'https://github.com/Madi-Toumani-benmohamed/Iac-Docker-Jenkins-build-push.git'
            }
        }
        stage('Test') {
            steps {
                sh './run-tests.sh' // Script pour exécuter les tests
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("bmaditoumani/web:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push to Docker Registry') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials-id') {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker-compose up -d'
            }
        }
    }
    post {
        success {
            echo 'Déploiement réussi.'
        }
        failure {
            echo 'Le build ou le déploiement a échoué.'
        }
    }
}
