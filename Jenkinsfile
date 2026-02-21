pipeline {
    agent none

    stages {
        stage('Clone, Install y Test') {
            agent {
                docker {
                    image 'python:3.12-slim'
                    args '-u root:root'
                }
            }
            stages {
                stage('Clone') {
                    steps {
                        git branch: 'master', url: 'https://github.com/daemoncibsec/django_tutorial.git'
                    }
                }
                stage('Install') {
                    steps {
                        sh 'pip install -r requirements.txt'
                    }
                }
                stage('Test') {
                    steps {
                        sh 'python3 manage.py test'
                    }
                }
            }
        }

        stage('Build Image') {
            agent { label 'built-in' }
            steps {
                git branch: 'master', url: 'https://github.com/daemoncibsec/django_tutorial.git'
                sh 'docker build -t daemoncibsec/app-django:latest .'
            }
        }

        stage('Push to Docker Hub') {
            agent { label 'built-in' }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push daemoncibsec/app-django:latest
                    '''
                }
            }
        }

        stage('Delete Local Image') {
            agent { label 'built-in' }
            steps {
                sh 'docker rmi daemoncibsec/app-django:latest'
            }
        }

        stage('Deploy to VPS') {
            agent { label 'built-in' }
            steps {
                sshagent(credentials: ['vps-ssh-credentials']) {
                    sh '''
                        ssh goliath.da3m0n.org "
                            cd jenkins &&
                            docker compose down &&
                            docker compose pull &&
                            docker compose up -d
                        "
                    '''
                }
            }
        }
    }

    post {
        always {
            mail to: 'daemoncibsec@gmail.com',
                 subject: "Pipeline ${currentBuild.fullDisplayName} - ${currentBuild.currentResult}",
                 body: """
                    Estado: ${currentBuild.currentResult}
                    Proyecto: ${env.JOB_NAME}
                    Build número: ${env.BUILD_NUMBER}
                    URL: ${env.BUILD_URL}
                 """
        }
    }
}
