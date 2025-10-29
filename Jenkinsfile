pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "ajayabd17/tasktimer-react"
    }

    stages {
        stage('Declarative: Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Checkout React App') {
            steps {
                dir('app') {
                    git branch: 'main', url: 'https://github.com/ajayabd17/TaskTimer-React.git'
                }
            }
        }

        stage('Build React App') {
            steps {
                dir('app') {
                    bat 'npm install'
                    bat 'npm run build'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def tag = "${env.BUILD_NUMBER}"
                    bat "docker build -t ${DOCKER_IMAGE}:${tag} ./app"
                    bat "docker tag ${DOCKER_IMAGE}:${tag} ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                        docker push ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Prepare New Color') {
            steps {
                script {
                    bat '''
                        kubectl get svc tasktimer-service -o=jsonpath="{.spec.selector.version}" > version.txt 2>nul || echo none > version.txt
                    '''

                    def current = readFile('version.txt').trim()
                    if (current == '' || current == 'none') { current = 'green' }
                    env.CURRENT_COLOR = current
                    env.NEW_COLOR = (current == 'blue') ? 'green' : 'blue'
                    echo "Current: ${env.CURRENT_COLOR}, Deploying: ${env.NEW_COLOR}"
                }
            }
        }

        stage('Blue-Green Deploy') {
            steps {
                bat "kubectl apply -f \"${WORKSPACE}\\kubernetes\\deployment-${env.NEW_COLOR}.yaml\""
                bat "kubectl rollout status deployment/tasktimer-${env.NEW_COLOR} --timeout=120s"
            }
        }

        stage('Switch Traffic') {
            steps {
                bat "kubectl apply -f \"${WORKSPACE}\\kubernetes\\service.yaml\""
                bat "kubectl patch service tasktimer-service -p \"{\\\"spec\\\":{\\\"selector\\\":{\\\"app\\\":\\\"tasktimer\\\",\\\"version\\\":\\\"${env.NEW_COLOR}\\\"}}}\""
            }
        }

        stage('Delete Old Deployment') {
            steps {
                script {
                    def deleteCmd = "kubectl delete deployment tasktimer-${env.CURRENT_COLOR} --ignore-not-found"
                    echo "Cleaning old: ${deleteCmd}"
                    bat deleteCmd
                }
            }
        }

        stage('Smoke Test') {
            steps {
                script {
                    def url = bat(script: "minikube service tasktimer-service --url | findstr http", returnStdout: true).trim()
                    echo "Testing: ${url}"
                    bat "curl -f ${url} || exit 1"
                }
            }
        }

        stage('Post Deployment Health Check') {
            steps {
                bat 'kubectl get pods -l app=tasktimer'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            echo 'Cleaning up...'
        }
    }
}