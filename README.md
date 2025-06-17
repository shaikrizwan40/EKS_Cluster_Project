# 🚀 EKS Fargate Demo with AWS ALB Ingress Controller

This project demonstrates deploying a sample application (`2048 game`) on an **Amazon EKS cluster using Fargate** and configuring it with **AWS ALB Ingress Controller** using **IRSA (IAM Roles for Service Accounts)**.

---

## **🧰 Pre-Requisites**
1. **kubectl** – Tool to manage Kubernetes cluster resources  
2. **awscli** – To interact with AWS CLI  
3. **eksctl** – To create and manage the EKS cluster

---

## **🔧 Step 1: Create the EKS Cluster using Fargate**
```
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```

---

## **🔄 Step 2: Update the kubeconfig**
```
aws eks update-kubeconfig --name demo-cluster --region us-east-1
```

---

## **📦 Step 3: Apply the application manifest**
```
kubectl apply -f https://raw.githubusercontent.com/shaikrizwan40/EKS_Cluster_Project/main/Manifest/circus-game-manifests.yaml
```

---

## **🔐 Step 4: Enable AWS Access via IRSA**

To allow pods to communicate securely with AWS services:

### a. **Associate IAM OIDC Provider with the EKS Cluster**
```
eksctl utils associate-iam-oidc-provider --cluster demo-cluster --approve
```

### b. **Create IAM Policy for AWS Load Balancer Controller**
```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam-policy.json
```

### c. **Create IAM Service Account for ALB Controller**
```
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn arn:aws:iam::<YOUR-AWS-ACCOUNT-ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --override-existing-serviceaccounts
```

---

## **🧱 Step 5: Install AWS Load Balancer Controller via Helm**

### a. **Add Helm Repo**
```
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

### b. **Install the Controller**
```
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set region=us-east-1 \
  --set vpcId=<YOUR-VPC-ID> \
  --set serviceAccount.name=aws-load-balancer-controller
```

> Replace `<YOUR-VPC-ID>` with your actual VPC ID and `<YOUR-AWS-ACCOUNT-ID>` above.

---

## **🌐 Step 6: Verify the Deployment**

### Check pods:
```
kubectl get pods -n kube-system

```

### Get Ingress:
```
kubectl get ingress --all-namespaces
```

Copy the **ALB DNS Name** and access it in your browser to verify the `2048 game` app is running.

---

## ✅ Troubleshooting Tips

- **Pods stuck in Pending?**  
  Ensure the correct Fargate profile exists for the namespace:
  ```
  eksctl get fargateprofile --cluster demo-cluster
  ```
  You must have a Fargate profile covering the namespace (`game-2048`) where the pods are deployed.

- **503 Service Unavailable?**  
  - Ensure your target pods are `Running` and `Ready`
  - Make sure your Service type is `NodePort` or `ClusterIP` and correctly referenced in the Ingress

---

## 👨‍💻 Author
**Shaik Mohammad Rizwan**  
DevOps Engineer | AWS | Terraform | Jenkins | Kubernetes  | Docker | GitHub | Ansible |Shell Scripting |
[LinkedIn Profile](https://www.linkedin.com/in/Md-Rizwan-shaik)
