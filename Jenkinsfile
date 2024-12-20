pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        SPRING_DATASOURCE_URL = 'jdbc:postgresql://localhost:5433/eBankifySecurity'
        DOCKER_IMAGE = 'banking-system'
        DOCKER_TAG = "${BUILD_NUMBER}"
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    deleteDir()
                    echo "Clonage du dépôt Git..."
                    bat '''
                        git clone -b main https://github.com/MaryemKhaoua/CI-CD-jenkins .
                        echo "Dépôt cloné avec succès."
                    '''
                }
            }
        }

        stage('Environment Check') {
            steps {
                bat '''
                    echo "Version de Git :"
                    git --version
                    echo "Branche Git actuelle :"
                    git branch --show-current
                    echo "Statut de Git :"
                    git status
                    echo "Version de Java :"
                    java -version
                    echo "Version de Javac :"
                    javac -version
                    echo "Contenu du répertoire de travail :"
                    dir
                '''
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Unit Tests') {
            steps {
                bat 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Code Quality Analysis') {
            steps {
                bat '''
                    mvn sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.token=${env.SONAR_TOKEN}
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat '''
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Manual Approval') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    input message: 'Déployer en production ?', ok: 'Procéder'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    try {
                        bat 'docker ps -a | findstr postgres_db && docker rm -f postgres_db'
                    } catch (Exception e) {
                        echo "Pas de conteneur postgres_db existant."
                    }
                    bat '''
                        echo "Déploiement des services avec Docker Compose..."
                        docker-compose up -d
                    '''
                }
            }
        }
    }

    post {
        success {
            mail to: 'maryem.khaoua@gmail.com',
                 subject: "Pipeline Success - eBankify",
                 body: "Le pipeline Jenkins s'est terminé avec succès !"
        }
        failure {
            mail to: 'maryem.khaoua@gmail.com',
                 subject: "Pipeline Failure - eBankify",
                 body: "Le pipeline Jenkins a échoué. Veuillez vérifier les logs."
        }
    }
}
