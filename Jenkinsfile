pipeline {
    agent {
        docker {
            image 'python:3.9-slim'
            args '-u root' // if you need to run as root (use only if needed)
        }
    }
    environment {
        APP_DIR = '/app'
        REPO_URL = 'https://github.com/blackkolly/dockerizing-jenkins.git'
        SONARQUBE_SERVER = 'sonarqube'
        SONAR_HOST_URL = 'http://98.66.213.81:9000' // Add SonarQube URL
        SONAR_AUTH_TOKEN = credentials('squ_dc26c2b6e3b740bdb2fe363751de6a892566043f') // Ensure you add this credential
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: env.REPO_URL
            }
        }
        stage('Build App') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'python manage.py migrate'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'python manage.py test'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(env.SONARQUBE_SERVER) {
                        sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=django_app \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_AUTH_TOKEN}
                        '''
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials-id') {
                        def appImage = docker.build("blackkolly/django_app:${env.BUILD_NUMBER}")
                        appImage.push()
                    }
                }
            }
        }
        stage('Deploy App') {
            steps {
                script {
                    sh 'docker-compose up -d'
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
