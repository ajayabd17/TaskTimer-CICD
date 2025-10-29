pipeline {
  agent any
  environment {
    DOCKER_IMAGE = "ajayabd17/tasktimer-react"
  }

  stages {
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

    stage('Docker Build & Push') {
      steps {
        script {
          def tag = env.BUILD_NUMBER
          bat "docker build -t ${DOCKER_IMAGE}:${tag} ./app"
          bat "docker tag ${DOCKER_IMAGE}:${tag} ${DOCKER_IMAGE}:latest"
          withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            bat "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin"
            bat "docker push ${DOCKER_IMAGE}:${tag}"
            bat "docker push ${DOCKER_IMAGE}:latest"
          }
        }
      }
    }

    stage('Blue-Green Deploy') {
      steps {
        echo "Performing Blue-Green deployment..."

        bat '''
          kubectl get svc tasktimer-service -o=jsonpath="{.spec.selector.version}" > version.txt 2>nul
          if errorlevel 1 echo none > version.txt
        '''

        def current = readFile('version.txt').trim()
        if (current == '' || current == 'none') current = 'green'
        def newVersion = (current == 'blue') ? 'green' : 'blue'

        bat "kubectl apply -f \"${WORKSPACE}\\kubernetes\\deployment-${newVersion}.yaml\""
        bat "kubectl rollout status deployment/tasktimer-${newVersion} --timeout=120s"
        bat "kubectl apply -f \"${WORKSPACE}\\kubernetes\\service.yaml\""
        bat "kubectl patch service tasktimer-service -p \"{\\\"spec\\\":{\\\"selector\\\":{\\\"app\\\":\\\"tasktimer\\\",\\\"version\\\":\\\"${newVersion}\\\"}}}\""
        bat "kubectl delete deployment tasktimer-${current} --ignore-not-found=true"
      }
    }
  }

  post {
    success { echo 'Deployment successful on Windows Jenkins Agent!' }
    failure { echo 'Deployment failed!' }
  }
}