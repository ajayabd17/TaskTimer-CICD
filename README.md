# TaskTimer CI/CD (Blue-Green) - Ready bundle

## What is included
- Dockerfile
- Jenkinsfile (Declarative pipeline)
- kubernetes/deployment-blue.yaml
- kubernetes/deployment-green.yaml
- kubernetes/service.yaml
- troubleshoot_jenkins_500.md
- PBL_Ending.md

## Quick Start (Local Minikube + Jenkins)
1. Ensure Docker Desktop, Minikube, kubectl are installed and running.
2. Start Minikube with Docker driver: `minikube start --driver=docker`
3. Ensure Jenkins has Docker access (if Jenkins runs in a container, run it with `-v /var/run/docker.sock:/var/run/docker.sock`).
4. Create Jenkins credentials:
   - ID: `dockerhub` (Username/Password for Docker Hub)
5. Create a multibranch or Pipeline job in Jenkins and point to your GitHub repo.
6. Add the provided Jenkinsfile to the repo root (or use the one in this bundle).
7. From Jenkins, run the pipeline. It will:
   - build the React app
   - build + push Docker image to Docker Hub
   - apply blue-green deployment on Kubernetes
8. Access the app:
   - If using Minikube, get a node port: `kubectl get svc tasktimer-service -o wide`
   - Open the cluster IP or use `minikube service tasktimer-service --url`

## Notes
- The Jenkinsfile expects `docker` and `kubectl` to be available on the Jenkins agent.
- If Jenkins is running inside Docker and shows HTTP 500 errors, see `troubleshoot_jenkins_500.md`.
