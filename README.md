DevOPs_Masterpiece-project - CI/CD Pipeline

Overview
This project automates the build, test, and deployment process using Jenkins. The application is built, tested, and deployed as a Docker container to a Kubernetes (K8s) cluster. The following diagram illustrates the architecture:
file:///C:/Users/manoh/Downloads/1_MPeHOvu294XzZ7ONfHBU4A.webp


Pipeline Flow
Developer commits code to the GitHub repository.
Jenkins triggers a build process via a Git webhook.
Jenkins builds the Docker image and pushes it to DockerHub.
Kubernetes (K8s) manifests are updated with the new Docker image tag.
The updated K8s manifests are synced to the K8s cluster using ArgoCD to deploy the new Docker container to the pods.


Jenkins Pipeline Stages
The Jenkins pipeline is defined in the Jenkinsfile and consists of the following stages:

Checkout Git: Clones the main branch of the project from GitHub.
Get Git Commit Hash: Retrieves the commit hash to ensure the latest changes are built.
Build & JUnit Test: Runs mvn clean install to build the application and run unit tests using Maven.
Docker Build: Builds a Docker image using the application's source code and tags it with the commit hash.
Docker Image Push: Pushes the Docker image to DockerHub and then removes the local image to free space.
Clone/Pull K8s Deployment Repo: Clones the K8s manifest repository if it does not exist, or pulls the latest changes if it does.
Update Deployment Manifest: Updates the K8s deployment manifest with the newly built Docker image.
Commit & Push Changes to Feature Branch: Commits the updated deployment manifest to the feature branch in GitHub.
Raise Pull Request: Creates a pull request on GitHub for the updated K8s deployment manifest.
Project Structure
bash
Copy code
.
├── Jenkinsfile              # CI/CD Pipeline configuration for Jenkins
├── Dockerfile               # Instructions for building the Docker image
├── deployment.yaml          # Kubernetes deployment manifest
├── pom.xml                  # Maven project configuration
└── src/                     # Application source code
Prerequisites
Before setting up the pipeline, ensure the following are in place:

Jenkins: Set up Jenkins with Docker and Maven installed.
DockerHub Account: A DockerHub repository to push the images.
GitHub: A GitHub repository to store the project code and deployment manifests.
Kubernetes Cluster: A K8s cluster to deploy the Docker container.
ArgoCD: Installed and configured to sync the GitHub repository containing the K8s manifests with the Kubernetes cluster.
Setting up the Pipeline
Create Jenkins Pipeline: Create a new pipeline in Jenkins, point it to the project's GitHub repository, and use the Jenkinsfile provided.
Configure Credentials:
docker-credentials-id: DockerHub credentials to push the Docker image.
GITHUB_TOKEN: GitHub token to commit changes and create a pull request.
Configure Webhook: Set up a GitHub webhook to trigger Jenkins builds on code commits.
Docker Image
The Docker image for this application is built using the following command in the pipeline:

bash
Copy code
docker build -t indalarajesh/spring-app1:${VERSION}-${GIT_COMMIT} .
The image is pushed to DockerHub using:

bash
Copy code
docker push indalarajesh/spring-app1:${VERSION}-${GIT_COMMIT}
Kubernetes Deployment
The deployment manifest (deployment.yaml) is updated with the new Docker image tag during the pipeline, and a pull request is raised for the same. Once the PR is merged, ArgoCD will sync the changes to the Kubernetes cluster, updating the pods with the new image.

Example Deployment Manifest (deployment.yaml)
yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spring-app1
  template:
    metadata:
      labels:
        app: spring-app1
    spec:
      containers:
      - name: spring-app1
        image: indalarajesh/spring-app1:${VERSION}-${GIT_COMMIT}
        ports:
        - containerPort: 8080
Contributing
To contribute to this project:

Fork the repository.
Create a new feature branch.
Make changes and commit them.
Push to your fork and submit a pull request.
