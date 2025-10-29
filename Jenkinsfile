pipeline {
  agent any
  environment {
    DOCKER_IMAGE = "ajayabd17/tasktimer-react"
    DOCKERHUB_CREDENTIALS = credentials('dockerhub')
  }

  stages {
    stage('Checkout React App') {
      steps {
        // Clone the React project into a subdirectory called 'app'
        dir('app') {
          git branch: 'main', url: 'https://github.com/ajayabd17/TaskTimer-React.git'
        }
      }
    }

    stage('Build React App') {
      steps {
        echo 'Installing dependencies and building React app...'
        dir('app') {
          bat 'npm install'
          bat 'npm run build'
        }
      }
    }

    stage('Docker Build & Push') {
      steps {
        script {
          def tag = "${env.BUILD_NUMBER}"
          echo "Building Docker image: ${DOCKER_IMAGE}:${tag}"
          bat "docker build -t ${DOCKER_IMAGE}:${tag} ./app"
          bat "docker tag ${DOCKER_IMAGE}:${tag} ${DOCKER_IMAGE}:latest"

          withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                            usernameVariable: 'DOCKER_USER',
                                            passwordVariable: 'DOCKER_PASS')]) {
            bat """
              echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
              docker push ${DOCKER_IMAGE}:${tag}
              docker push ${DOCKER_IMAGE}:latest
            """
          }
        }
      }
    }

    stage('Blue-Green Deploy') {
      steps {
        script {
          echo 'Performing Blue-Green deployment on Kubernetes...'
          def svc = bat(script: "kubectl get svc tasktimer-service -o=jsonpath='{.spec.selector.version}' || echo none", returnStdout: true).trim()
          if (svc == 'none' || svc == '') { svc = 'green' }
          def newVersion = svc == 'blue' ? 'green' : 'blue'
          echo "Current svc version: ${svc}, deploying new version: ${newVersion}"

          // Adjust to your Jenkins workspace path for Kubernetes YAMLs
          bat "kubectl apply -f ${WORKSPACE}\\kubernetes\\deployment-${newVersion}.yaml"
          bat "kubectl rollout status deployment/tasktimer-${newVersion} --timeout=120s"
          bat "kubectl apply -f ${WORKSPACE}\\kubernetes\\service.yaml"
          bat "kubectl patch service tasktimer-service -p \"{\\\"spec\\\":{\\\"selector\\\":{\\\"app\\\":\\\"tasktimer\\\",\\\"version\\\":\\\"${newVersion}\\\"}}}\""

          if (svc != '') {
            bat "kubectl delete deployment tasktimer-${svc} || echo Old deployment already removed."
          }
        }
      }
    }
  }

  post {
    success { echo '✅ Deployment successful on Windows Jenkins Agent!' }
    failure { echo '❌ Deployment failed!' }
  }
}
