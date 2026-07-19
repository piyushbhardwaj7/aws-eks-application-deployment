Application Deployment on AWS EKS using Kubernetes

Deployed a containerized Flask backend application on Amazon Elastic Kubernetes Service (EKS). Built the Kubernetes cluster using eksctl, containerized the application with Docker, pushed the image to Amazon ECR, and deployed it using Kubernetes manifests.

Architecture

EC2 (Admin/Bastion Instance) → eksctl → Amazon EKS Cluster (Control Plane + Worker Nodes) → Docker Image → Amazon ECR (Container Registry) → Kubernetes Deployment (3 replicas) → Pods → Kubernetes Service (NodePort) → Exposes app on port 30080

Project Overview


Launched an EC2 instance to serve as the admin/bastion host for managing the EKS cluster
Created an IAM role and attached it to the EC2 instance to allow it to call AWS services
Installed and configured eksctl and kubectl on the instance
Provisioned an Amazon EKS cluster with a managed worker node group using eksctl
Cloned a Flask backend application, containerized it with Docker
Pushed the Docker image to a private Amazon ECR repository
Wrote Kubernetes Deployment and Service manifests
Deployed the application to the EKS cluster and verified all pods were running


Technologies Used

AWS, Amazon EKS, Amazon EC2, Amazon ECR, IAM, Kubernetes, Docker, kubectl, eksctl, Git & GitHub, Flask (Python)

Project Workflow

1. Provisioned the Admin EC2 Instance


Launched a t2.micro EC2 instance (Amazon Linux 2023) to act as the management host
Connected via EC2 Instance Connect


2. Configured IAM Access


Created an IAM role and attached it to the EC2 instance so it could interact with AWS services (EKS, ECR) on my behalf
Verified the role was correctly attached under Instance Settings


3. Installed Cluster Tooling


Installed eksctl on the EC2 instance
Installed kubectl and verified the client version


4. Created the EKS Cluster

Ran the following command to provision the EKS control plane and a managed node group across multiple availability zones in ap-south-1:

eksctl create cluster --name piyush-eks-cluster1 --region ap-south-1 --nodegroup-name worker-nodes --nodes 2 --nodes-min 1 --nodes-max 3 --node-type t3.micro

5. Prepared the Application


Installed Git and Docker on the EC2 instance
Added ec2-user to the docker group to run Docker without sudo
Cloned the Flask backend application repository
Reviewed the Dockerfile and application structure (app.py, requirements.txt)


6. Containerized and Pushed to ECR


Built the Docker image locally on the EC2 instance
Created a private ECR repository with image scanning enabled on push
Authenticated Docker to ECR and pushed the built image


7. Wrote Kubernetes Manifests


Created flask-deployment.yaml defining a Deployment with 3 replicas, referencing the image in ECR
Created flask-service.yaml defining a NodePort Service to expose the application


8. Deployed to the Cluster

Applied the manifests and verified successful rollout using kubectl apply -f flask-deployment.yaml and kubectl apply -f flask-service.yaml, then checked status with kubectl get pods.

All 3 replica pods reached Running status with 0 restarts, confirming a successful deployment:

NAME                                       READY   STATUS    RESTARTS   AGE
nextwork-flask-backend-67f9cdbb5-ft88d     1/1     Running   0          3m49s
nextwork-flask-backend-67f9cdbb5-kjcmk     1/1     Running   0          3m49s
nextwork-flask-backend-67f9cdbb5-w2wzr     1/1     Running   0          3m49s

Kubernetes Resources Created

Deployment, Pods (3 replicas), ReplicaSet, Service (NodePort)

AWS Resources Used

Amazon EKS Cluster, EC2 Admin Instance, EC2 Worker Nodes (managed node group), Amazon ECR Repository, IAM Role, VPC, Security Groups

Commands Used

eksctl and kubectl setup:
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)amd64.tar.gz"
tar -xzf eksctl$(uname -s)_amd64.tar.gz
sudo mv eksctl /usr/local/bin/
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

Cluster creation:
eksctl create cluster --name piyush-eks-cluster1 --region ap-south-1 --nodegroup-name worker-nodes --nodes 2 --nodes-min 1 --nodes-max 3 --node-type t3.micro

Docker build and ECR push:
docker build -t flask-backend .
aws ecr create-repository --repository-name flask-backend --image-scanning-configuration scanOnPush=true
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.ap-south-1.amazonaws.com
docker tag flask-backend:latest <account-id>.dkr.ecr.ap-south-1.amazonaws.com/flask-backend:latest
docker push <account-id>.dkr.ecr.ap-south-1.amazonaws.com/flask-backend:latest

Deployment:
kubectl apply -f flask-deployment.yaml
kubectl apply -f flask-service.yaml
kubectl get pods
kubectl get deployments
kubectl get services

Key Concepts Practiced

Cluster, Node, Pod, Deployment, ReplicaSet, Service (NodePort), Labels & Selectors, IAM Roles, Container Images, YAML Manifests, Desired State

What I Learned


How to provision an EKS cluster end-to-end using eksctl, including the managed node group
How IAM roles attached to EC2 instances grant permissions to interact with other AWS services (EKS, ECR)
How to containerize a Python/Flask application and manage its lifecycle through Amazon ECR
How Kubernetes Deployments maintain desired state and manage ReplicaSets/Pods
The difference between Service types (chose NodePort here) and how selectors connect Services to Pods
How to verify successful pod scheduling and rollout using kubectl get pods
