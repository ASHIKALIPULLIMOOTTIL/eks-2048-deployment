# eks-2048-deployment
Deploying a Kubernetes application on AWS EKS using Fargate and ALB Ingress.
# AWS EKS 2048 Deployment

This project demonstrates deployment of a containerized 2048 application on Amazon EKS using Kubernetes, Fargate, and AWS Load Balancer Controller.

---

## 🚀 Technologies Used

- AWS EKS
- Kubernetes
- AWS Fargate
- AWS Load Balancer Controller
- kubectl
- eksctl
- Helm
- AWS IAM
- AWS VPC

---

## 📌 Architecture

Internet → ALB → Ingress → Service → Pods → Fargate

---

## 📌 Features

- Serverless Kubernetes deployment using Fargate
- ALB Ingress integration
- Kubernetes Deployments and Services
- Scalable container orchestration

---

## 📌 Deployment Steps

### Create EKS Cluster

```bash
eksctl create cluster --name demo-cluster --region ap-south-1 --fargate
```

### Deploy Application

```bash
kubectl apply -f manifests/2048_full.yaml
```

---
## 📌 Learning Outcome

- Hands-on experience with Amazon EKS
- Kubernetes resource management
- Ingress and Load Balancer configuration
- Fargate-based deployments
