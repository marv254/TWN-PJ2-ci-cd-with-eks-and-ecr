# Complete CI/CD Pipeline with EKS and AWS ECR

**Technologies used:**

Kubernetes, Jenkins, AWS EKS, AWS ECR, Java, Maven, Linux, Docker, Git

**Project Description**

- Created private AWS ECR Docker repository
- Configured Jenkinsfile to build and push Docker Image to AWS ECR
- Integrated deploying to K8s cluster in the CI/CD pipeline from AWS ECR private registry

So the complete CI/CD project has the following configuration:
* CI step: Increment version
* CI step: Build artifact for Java Maven application
* CI step: Build and push Docker image to AWS ECR
* CD step: Deploy new application version to EKS cluster
* CD step: Commit the version update
