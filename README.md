# EKS_Cluster_Project
Code for Amazon EKS Cluster Project Using Fargate from Scratch

Pre-Requisites:
1. kubectl - Tool to manage the Kuberntes Cluster resources.
2. awscli  -  To connect with AWS
3. eksctl  - To create and manage the EKS cluster


Step1: Create the EKS Cluster using Fargate
eksctl create cluster --name demo-cluster --region us-east-1 --fargate

Step2: Update the kube-config file
aws eks update-kubeconfig --name demo-cluster --region us-east-1

Step3: Apply the Manifest files.
For Demo Files, apply the command

kubectl apply -f 
