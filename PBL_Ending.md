# PBL - III: DevOps — Final Section (Blue-Green Deployment Automation)

**Project:** CI/CD Pipeline for TaskTimer-React (Blue-Green Deployment)  
**Student:** Ajay Dev Pavithran (ajayabd17)  
**Supervisor:** [Your Supervisor Name]  
**Date:** 29-10-2025

## Conclusion and Outcome

This project implemented an automated CI/CD pipeline using Jenkins that builds a Dockerized React application, pushes images to Docker Hub, and performs a zero-downtime Blue-Green deployment on a Kubernetes cluster (Minikube). The pipeline automates:

- Source checkout from GitHub
- Build and packaging of the frontend using a multi-stage Dockerfile
- Image publishing to Docker Hub
- Deployment to Kubernetes using separate Blue and Green deployments
- Service switching (traffic cutover) to the newly validated version

The resulting process ensures rapid, repeatable, and safer releases with minimal downtime and easy rollback by retaining the previous deployment until it is deleted.

## Future Work & Improvements

- Add automated health checks and canary analysis before traffic cutover.
- Integrate image scanning and vulnerability checks before pushing.
- Implement GitHub webhooks to trigger Jenkins on push.
- Use Helm charts and Kubernetes Ingress for production-grade routing and TLS.

## How the work ends

The CI/CD pipeline is delivered as a deployable bundle. To complete evaluation:
1. Configure Jenkins credentials and ensure Jenkins agent can access Docker and kubectl.
2. Run the pipeline — verify that new versions are deployed using the Blue-Green strategy.
3. Document any runtime issues and attach logs for evaluation.

**Project Status:** Delivered — Blue-Green pipeline implemented, ready for instructor verification.
