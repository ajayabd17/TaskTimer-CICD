# Troubleshooting Jenkins HTTP 500 (Common fixes)

If Jenkins UI responds with HTTP 500 or pipelines fail with server errors, try the following:

1. Check Jenkins logs (inside container): 
   - If Jenkins runs in Docker: `docker logs <jenkins-container-id> --tail 200`
   - On system service: check `/var/log/jenkins/jenkins.log`
2. Disk space / Java heap issues:
   - Ensure container/host has enough disk and memory. Jenkins may crash if no disk.
   - Increase Java opts: `JENKINS_JAVA_OPTS=-Xmx1024m`
3. Plugin incompatibility:
   - A bad plugin can cause 500s. Start Jenkins in safe mode: `http://<host>:8080/safeRestart`
   - Remove recently installed plugins.
4. File permission or workspace corruption:
   - Clean up workspace directories and restart Jenkins.
5. Reverse proxy / Nginx / Traefik misconfiguration:
   - If using a proxy, ensure headers and timeouts are forwarded properly.
6. Check Docker-in-Docker / socket permission:
   - If Jenkins needs Docker, ensure `docker.sock` is mounted and Jenkins user can access it.
7. Recreate container:
   - Backup `JENKINS_HOME` then recreate container to restore a healthy state.
