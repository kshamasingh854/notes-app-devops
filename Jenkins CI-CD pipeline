
pipeline {
    agent {
        label 'kshsingh'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'

                git branch: 'main',
                    credentialsId: 'github-private',
                    url: 'https://github.com/kshamasingh854/django-notes-app.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building Docker image'

                sh '''
                    docker build -t django-notes-app:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Testing Docker image'

                sh '''
                    docker images django-notes-app:${BUILD_NUMBER}
                    docker image inspect django-notes-app:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing Docker image to Docker Hub'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-passwd',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker tag django-notes-app:${BUILD_NUMBER} \
                            $DOCKER_USER/django-notes-app:${BUILD_NUMBER}

                        docker tag django-notes-app:${BUILD_NUMBER} \
                            $DOCKER_USER/django-notes-app:latest

                        docker push $DOCKER_USER/django-notes-app:${BUILD_NUMBER}
                        docker push $DOCKER_USER/django-notes-app:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Kubernetes Connection Test') {
            steps {
                echo 'Testing Kubernetes connection'

                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        kubectl get deployments
                        kubectl get pods
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying application to Kubernetes'

                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    withEnv(["IMAGE=${env.DOCKER_USER ?: 'kshamasingh854'}/django-notes-app:${env.BUILD_NUMBER}"]) {
                        sh '''
                            kubectl set image deployment/notes-app \
                                notes-app=$IMAGE

                            kubectl rollout status deployment/notes-app \
                                --timeout=180s
                        '''
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying Kubernetes deployment'

                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        kubectl get deployments
                        kubectl get pods -o wide
                        kubectl get services
                    '''
                }
            }
        }
    }
}
