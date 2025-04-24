pipeline {
    agent {
        docker {
            image 'python:3.9-slim'
            // Remove -u root if possible (add jenkins user to docker group instead)
            args '-v /var/run/docker.sock:/var/run/docker.sock' 
        }
    }
    
    environment {
        APP_DIR = '/app'
        REPO_URL = 'https://github.com/blackkolly/dockerizing-jenkins.git'
        DOCKER_IMAGE = 'blackkolly/django_app'
        // Choose ONE SonarQube approach:
        // EITHER use withSonarQubeEnv:
        SONARQUBE_SERVER = 'sonarqube' 
        // OR use direct URL:
        // SONAR_HOST_URL = 'http://98.66.213.81:9000'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: env.REPO_URL
            }
        }
        
        stage('Build App') {
            steps {
                sh 'pip install --no-cache-dir -r requirements.txt'
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
                withSonarQubeEnv(env.SONARQUBE_SERVER) {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=django_app \
                    -Dsonar.sources=. \
                    -Dsonar.python.version=3.9
                    '''
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials-id') {
                        def appImage = docker.build("${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                        appImage.push()
                        // Tag as latest
                        appImage.push('latest')
                    }
                }
            }
        }
        
        stage('Deploy App') {
            steps {
                script {
                    // Use specific version rather than latest for production
                    sh "docker-compose pull"
                    sh "docker-compose up -d --no-deps --build web"
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
            script {
                // Clean up dangling images
                sh 'docker system prune -f || true'
            }
        }
        success {
            slackSend(color: 'good', message: "Pipeline SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
        failure {
            slackSend(color: 'danger', message: "Pipeline FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
    }
}
          
