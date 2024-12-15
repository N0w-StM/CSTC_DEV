Start Jenkins and SonarQube:
docker-compose up -d

docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword

Step 3: Configure GitHub Secrets
Store the following secrets in your GitHub repository:

bash
Copy code
# Set Docker Hub credentials
gh secret set DOCKER_USERNAME --body "your-dockerhub-username"
gh secret set DOCKER_PASSWORD --body "your-dockerhub-password"

# Set SonarQube token
gh secret set SONAR_TOKEN --body "your-sonarqube-token"

# Set Checkmarx API token
gh secret set CHECKMARX_TOKEN --body "your-checkmarx-api-token"

# Set Kubernetes config (base64-encoded)
cat ~/.kube/config | base64 | gh secret set KUBECONFIG

Step 4: Build and Deploy the Akaunting Application
Build and tag the Docker image locally:
bash
Copy code
docker build -t your-dockerhub-username/akaunting -f Dockerfiles/Dockerfile.akaunting .
Push the image to Docker Hub:
bash
Copy code
docker login
docker push your-dockerhub-username/akaunting:latest
Deploy to Kubernetes:
bash
Copy code
kubectl apply -f k8s/
Verify that the deployment is running:
bash
Copy code
kubectl get pods
kubectl get services
Access the Akaunting app through the LoadBalancer IP:

bash
Copy code
kubectl get svc akaunting-service
Step 5: Jenkins Pipeline Execution
Open Jenkins (http://localhost:8080).
Create a new Pipeline Job.
In the job configuration:
Select Pipeline Script from SCM.
Provide the GitHub repository URL.
Save and run the pipeline.
Step 6: SonarQube and Checkmarx Analysis
SonarQube:

Verify results at http://localhost:9000 using your SonarQube token.
Checkmarx:

Ensure your GitHub Actions workflow (checkmarx-analysis.yml) runs successfully.
Step 7: GitHub Actions Workflow
Push your changes to GitHub to trigger the workflows:

bash
Copy code
git add .
git commit -m "Add CI/CD pipeline setup"
git push origin main
The CI/CD pipeline will:
Build the Docker image.
Push it to Docker Hub.
Deploy the app to Kubernetes.
Perform SonarQube and Checkmarx analysis.
Step 8: Verify the App
Access the deployed Akaunting app via the Kubernetes LoadBalancer.
Check SonarQube results for code quality.
Verify Checkmarx analysis for security vulnerabilities.
Summary
Docker Compose: Runs Jenkins, SonarQube, and PostgreSQL locally.
Jenkins Pipeline: Automates build, push, and deployment.
GitHub Actions: CI/CD workflows for automation, code quality, and security scans.
Kubernetes: Deploys the Akaunting app.
Secrets Management: Uses GitHub Secrets and Kubernetes Sealed Secrets.
This setup provides a robust, automated CI/CD pipeline with code analysis and security scanning.

Let me know if you need further clarification or assistance! 🚀

