pipeline {
  agent any
  environment {
    DOCKER_IMAGE = "ajayabd17/tasktimer-react"
    DOCKERHUB_CREDENTIALS = credentials('dockerhub') // set this credential in Jenkins (username/password)
  }
  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/ajayabd17/TaskTImer-React.git'
      }
    }
    stage('Build') {
      steps {
        sh 'npm install'
        sh 'npm run build'
      }
    }
    stage('Docker Build & Push') {
      steps {
        script {
          def tag = "${env.BUILD_NUMBER}"
          sh "docker build -t ${DOCKER_IMAGE}:${tag} ."
          sh "docker tag ${DOCKER_IMAGE}:${tag} ${DOCKER_IMAGE}:latest"
          withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh '''
              echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
              docker push ${DOCKER_IMAGE}:${tag}
              docker push ${DOCKER_IMAGE}:latest
            '''
          }
        }
      }
    }
    stage('Blue-Green Deploy') {
      steps {
        script {
          // determine current version
          def svc = sh(script: "kubectl get svc tasktimer-service -o=jsonpath='{.spec.selector.version}' || true", returnStdout: true).trim()
          if (svc == '') { svc = 'green' } // default if service not present
          def newVersion = svc == 'blue' ? 'green' : 'blue'
          echo "Current svc version: ${svc}, deploying new version: ${newVersion}"
          sh "kubectl apply -f kubernetes/deployment-${newVersion}.yaml"
          sh "kubectl rollout status deployment/tasktimer-${newVersion} --timeout=120s"
          // switch service selector
          sh "kubectl apply -f kubernetes/service.yaml"
          sh "kubectl patch service tasktimer-service -p '{\"spec\":{\"selector\":{\"app\":\"tasktimer\",\"version\":\"${newVersion}\"}}}'"
          // optional: remove old deployment after switch
          if (svc != '') {
            sh "kubectl delete deployment tasktimer-${svc} || true"
          }
        }
      }
    }
  }
  post {
    success { echo 'Deployment successful' }
    failure { echo 'Deployment failed' }
  }
}
