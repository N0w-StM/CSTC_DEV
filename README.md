:doctype: book
:doctitle: Akaunting CI/CD Pipeline Setup
:docdate: June 8, 2024
:author: @ABDO_AB
:email: abidoabdi71@.com
:revnumber: 1.0.0
:revdate: June 8, 2024
:revremark: Initial version of CI/CD pipeline setup
:description: CI/CD pipeline setup for Akaunting using Jenkins, SonarQube, Kubernetes, and GitHub Actions.
:summary: This document explains how to set up a robust CI/CD pipeline for the Akaunting project.
:library: Asciidoctor
:source-highlighter: highlight.js
:keywords: akaunting, ci/cd, jenkins, sonarqube, kubernetes, github actions
:sectanchors:
:sectlinks:
:sectnums:
:toc:

*Author* {author} [{email}] +
*Date* {docdate} *Revision* {revnumber}, {revdate} +
*Tags* {keywords}

[NOTE]
This document outlines the steps for setting up a complete CI/CD pipeline for the Akaunting application.

== Prerequisites

To set up and run this pipeline, ensure you have the following tools installed:

- Docker and Docker Compose
- Kubernetes Cluster (Minikube, K3s, AWS EKS, GKE, AKS, or others)
- kubectl (Kubernetes CLI)
- GitHub CLI (`gh`)
- SonarQube Scanner
- Checkmarx API Token
- Access to a Docker Hub account for image hosting

== Folder Structure

The project is organized as follows:

[source,plaintext]
----
akaunting-ci-cd/
├── Dockerfiles/
│   ├── Dockerfile.jenkins       # Jenkins Dockerfile
│   ├── Dockerfile.sonarqube     # SonarQube Dockerfile
│   ├── Dockerfile.akaunting     # Akaunting Application Dockerfile
│
├── k8s/
│   ├── deployment.yaml          # Kubernetes Deployment
│   ├── service.yaml             # Kubernetes Service
│   ├── sealed-secret.yaml       # Kubernetes Sealed Secret
│
├── .github/
│   └── workflows/
│       ├── ci-cd.yml            # CI/CD GitHub Actions Workflow
│       ├── sonar-analysis.yml   # SonarQube Workflow
│       └── checkmarx-analysis.yml  # Checkmarx Workflow
│
├── Jenkinsfile                  # Jenkins Pipeline Script
├── sonar-project.properties     # SonarQube Project Configuration
├── README.md                    # Project Documentation
└── src/                         # Akaunting Source Code
----

== Setup Instructions

=== Step 1: Clone the Repository

Clone the repository from GitHub:

[source,bash]
----
git clone https://github.com/your-username/akaunting.git
cd akaunting
----

=== Step 2: Build and Run Jenkins and SonarQube

1. Create a `docker-compose.yml` file with the following content:

[source,yaml]
----
version: '3.8'
services:
  jenkins:
    build: Dockerfiles/Dockerfile.jenkins
    container_name: jenkins
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_data:/var/jenkins_home

  sonarqube:
    build: Dockerfiles/Dockerfile.sonarqube
    container_name: sonarqube
    ports:
      - "9000:9000"
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar
      - SONAR_JDBC_USERNAME=sonar
      - SONAR_JDBC_PASSWORD=sonar

  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
      - POSTGRES_DB=sonar
    volumes:
      - postgresql_data:/var/lib/postgresql/data

volumes:
  jenkins_data:
  postgresql_data:
----

2. Start the services:

[source,bash]
----
docker-compose up -d
----

3. Access the services:
- Jenkins: http://localhost:8080
- SonarQube: http://localhost:9000

Retrieve the Jenkins admin password:

[source,bash]
----
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
----

=== Step 3: Configure GitHub Secrets

Set the following secrets in your GitHub repository:

[source,bash]
----
gh secret set DOCKER_USERNAME --body "your-dockerhub-username"
gh secret set DOCKER_PASSWORD --body "your-dockerhub-password"
gh secret set SONAR_TOKEN --body "your-sonarqube-token"
gh secret set CHECKMARX_TOKEN --body "your-checkmarx-token"
cat ~/.kube/config | base64 | gh secret set KUBECONFIG
----

=== Step 4: Build and Deploy Akaunting

1. Build the Docker image and push it to Docker Hub:

[source,bash]
----
docker build -t your-dockerhub-username/akaunting -f Dockerfiles/Dockerfile.akaunting .
docker login
docker push your-dockerhub-username/akaunting:latest
----

2. Deploy the application to Kubernetes:

[source,bash]
----
kubectl apply -f k8s/
----

3. Verify the deployment:

[source,bash]
----
kubectl get pods
kubectl get services
----

=== Step 5: Jenkins Pipeline Execution

1. Access Jenkins at http://localhost:8080.
2. Create a new **Pipeline Job**:
   - Select **Pipeline Script from SCM**.
   - Provide your GitHub repository URL.
3. Run the pipeline.

=== Step 6: SonarQube and Checkmarx Analysis

- **SonarQube**: View the analysis results at http://localhost:9000.
- **Checkmarx**: Verify the code security scan in your GitHub Actions workflow.

=== Step 7: GitHub Actions Workflow

Push changes to trigger the GitHub Actions workflows:

[source,bash]
----
git add .
git commit -m "Add CI/CD pipeline setup"
git push origin main
----

=== Step 8: Verify the Application

1. Retrieve the **LoadBalancer IP** of the application:

[source,bash]
----
kubectl get svc akaunting-service
----

2. Access the Akaunting app in your browser:

[source,plaintext]
----
http://<LoadBalancer-IP>
----

== Technologies Used

- **Docker** and **Docker Compose**: Containerization of services.
- **Jenkins**: CI/CD automation.
- **SonarQube**: Code quality analysis.
- **Checkmarx**: Security vulnerability scanning.
- **Kubernetes**: Application orchestration and deployment.
- **GitHub Actions**: CI/CD workflows for automation.
- **Sealed Secrets**: Secure management of Kubernetes secrets.

== Conclusion

This setup provides a secure and automated CI/CD pipeline for the Akaunting project, integrating code quality, security scanning, and deployment automation.

Feel free to contribute or raise issues! 🚀

== References

- link:https://akaunting.com[Official Akaunting Website]
- link:https://kubernetes.io[Kubernetes Documentation]
- link:https://www.sonarqube.org[SonarQube Documentation]
- link:https://checkmarx.com[Checkmarx Documentation]
- link:https://github.com/actions[GitHub Actions]

