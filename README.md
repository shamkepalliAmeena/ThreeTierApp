# #ThreeTierApp

**Project Title:**

Deploying a Three-Tier Web Application using ReactJS, NodeJS, and MongoDB on AWS EKS and  through IAM user setup, infrastructure provisioning, CI/CD pipeline configuration, EKS cluster creation, and more.


**Project Overview:**

This project involves designing, containerizing, and deploying a three-tier web application consisting of a ReactJS frontend, a NodeJS backend, and a MongoDB database, hosted on Amazon Elastic Kubernetes Service (EKS). The goal is to build a scalable, fault-tolerant, and automated deployment architecture following modern DevOps and cloud-native practices.

**Architecture Overview:**

- Frontend (Presentation Layer): Developed using ReactJS, providing a responsive and interactive UI.

- Backend (Application Layer): Implemented using NodeJS and Express, exposing REST APIs for communication between the frontend and database.

- Database (Persistence Layer): MongoDB is used for storing and managing application data.

Each component is containerized using Docker and orchestrated using Kubernetes on AWS EKS for scalability and resilience.

## Prerequisites
- Basic knowledge of Docker, and AWS services.
- An AWS account with necessary permissions.

## Challenge Steps
- [Application Code](#application-code)
- [Jenkins Pipeline Code](#jenkins-pipeline-code)
- [Jenkins Server Terraform](#jenkins-server-terraform)
- [Kubernetes Manifests Files](#kubernetes-manifests-files)
- [Project Details](#project-details)

## Application Code
The `Application-Code` directory contains the source code for the Three-Tier Web Application. Dive into this directory to explore the frontend and backend implementations.

## Jenkins Pipeline Code
In the `Jenkins-Pipeline-Code` directory, you'll find Jenkins pipeline scripts. These scripts automate the CI/CD process, ensuring smooth integration and deployment of your application.

## Jenkins Server Terraform
Explore the `Jenkins-Server-TF` directory to find Terraform scripts for setting up the Jenkins Server on AWS. These scripts simplify the infrastructure provisioning process.

## Kubernetes Manifests Files
The `Kubernetes-Manifests-Files` directory holds Kubernetes manifests for deploying your application on AWS EKS. Understand and customize these files to suit your project needs.

## Project Details
🛠️ **Tools Explored:**
- Terraform & AWS CLI for AWS infrastructure
- Jenkins, Sonarqube, Terraform, Kubectl, and more for CI/CD setup
- Helm, Prometheus, and Grafana for Monitoring
- ArgoCD for GitOps practices

🚢 **High-Level Overview:**
- IAM User setup & Terraform magic on AWS
- Jenkins deployment with AWS integration
- EKS Cluster creation & Load Balancer configuration
- Private ECR repositories for secure image management
- Helm charts for efficient monitoring setup
- GitOps with ArgoCD - the cherry on top!

## Getting Started

### Step 1: IAM Configuration
- Create a user `eks-admin` with `AdministratorAccess`.
- Generate Security Credentials: Access Key and Secret Access Key.

### Step 2: EC2 Setup
- Launch an Ubuntu instance in your favourite region (eg. region `ap-south-1`).
- SSH into the instance from your local machine.

### Step 3: Install AWS CLI v2
``` shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```

### Step 4: Install Docker
``` shell
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```

### Step 5: Install kubectl
``` shell
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### Step 6: Install eksctl
``` shell
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Step 7: Setup EKS Cluster
``` shell
eksctl create cluster --name three-tier-cluster --region ap-south-1 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region ap-south-1 --name three-tier-cluster
kubectl get nodes
```

### Step 8: Run Manifests
``` shell
kubectl create namespace workshop
kubectl apply -f .
kubectl delete -f .
```

### Step 9: Install AWS Load Balancer
``` shell
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=ap-south-1 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::605134458989:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=ap-south-1
```

### Step 10: Deploy AWS Load Balancer Controller
``` shell
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl apply -f ingress.yaml
```
**Three-Tier application is already deployed and accessible on EKS, you can now proceed to set up Jenkins for CI/CD.**

You can use:
- Same EC2 where you have AWS CLI, kubectl, and Docker ✅
- Separate EC2 (recommended for production, optional for now)
 
**Install Java (Required by Jenkins)**
``` shell 
sudo apt-get update -y
sudo apt-get install openjdk-17-jdk -y
java -version
```
**Install Jenkins**
``` shell 
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
```
**Start Jenkins**
``` shell 
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
``` 
Check it is running: active (running)
**Configure Security Group**

- Open port 8080 in your EC2 Security Group:

- Type: Custom TCP

- Port Range: 8080

- Source: Your IP (or 0.0.0.0/0 for testing)

  **Access Jenkins**
Go to:
``` shell

http://<EC2-Public-IP>:8080
```
* Copy initial password:
``` shell
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
``` 
* Install suggested plugins
* Create admin user

**Install Required Plugins**

* Pipeline
* GitHub Integration
* Docker Pipeline
* AWS Credentials
* Kubernetes CLI
* Blue Ocean (optional)

**8. Configure Jenkins Credentials**
Add:
* 1.AWS Credentials (for ECR & EKS)
* 2.GitHub Credentials (if repo private)
* 3.Docker credentials (optional if using ECR)

**9. Create Jenkins Pipeline Job**

* 1.New Item → Pipeline
* 2.Name: three-tier-eks-deploy
* 3.Pipeline → Pipeline script from SCM
* 4.SCM: Git → Repository URL → Branch = main → Script Path = Jenkinsfile

**Add Jenkinsfile**
``` shell
Use this (updated for your already deployed cluster):
pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '605134458989'.dkr.ecr.ap-south-1.amazonaws.com'
        FRONTEND_REPO = 'public.ecr.aws/e2c0t2g7/three-tier-frontend:latest'
        BACKEND_REPO = 'public.ecr.aws/e2c0t2g7/three-tier-backend:latest'
        KUBE_NAMESPACE = 'workshop'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/shamkepalliAmeena/three-tier-eks-deployment.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $public.ecr.aws/e2c0t2g7/three-tier-frontend:latest ./Application-Code/frontend'
                sh 'docker build -t $public.ecr.aws/e2c0t2g7/three-tier-backend:latest ./Application-Code/backend'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 605134458989'.dkr.ecr.ap-south-1.amazonaws.com'
            }
        }

        stage('Push Docker Images to ECR') {
            steps {
                sh 'docker push $public.ecr.aws/e2c0t2g7/three-tier-frontend:latest'
                sh 'docker push $public.ecr.aws/e2c0t2g7/three-tier-backend:latest'
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                aws eks update-kubeconfig --region ap-south-1 --name three-tier-cluster
                kubectl apply -f Kubernetes-Manifests-Files/
                kubectl rollout status deployment/frontend -n workshop
                kubectl rollout status deployment/backend -n workshop
                '''
            }
        }
    }

    post {
        success { echo '🎉 Deployment Successful!' }
        failure { echo '❌ Deployment Failed!' }
    }
}

```
**Build & Verify**

Click Build Now in Jenkins
**Watch each stage:**
* Code checkout 
* Docker build 
* Push to ECR 
* Deployment to EKS 

**Verify pods:**
``` shell
kubectl get pods -n workshop
kubectl get svc -n workshop
```
Everything should remain accessible, and Jenkins will handle new deployments automatically.

**### Cleanup**
- To delete the EKS cluster:
``` shell
eksctl delete cluster --name three-tier-cluster --region ap-south-1
```
- To clean up rest of the stuff and not incure any cost
```
Stop or Terminate the EC2 instance created in step 2.
Delete the Load Balancer created in step 9 and 10.
Go to EC2 console, access security group section and delete security groups created in previous steps.
remove ECR Repositories.
```

---
Happy Learning! 🚀👨‍💻👩‍💻
