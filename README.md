# AWS EKS Kubernetes Deployment — 2048 Game on Fargate with ALB Ingress

![AWS](https://img.shields.io/badge/AWS-EKS-orange?logo=amazon-aws)
![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.34-blue?logo=kubernetes)
![Fargate](https://img.shields.io/badge/Compute-AWS%20Fargate-purple)
![Helm](https://img.shields.io/badge/Helm-v3.20.0-blue?logo=helm)
![License](https://img.shields.io/badge/License-MIT-green)

A production-style Kubernetes deployment on **AWS EKS with Fargate**, exposing the classic 2048 game via an **AWS Application Load Balancer (ALB)** using the AWS Load Balancer Controller and Helm.

---

## Table of Contents

- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Deployment Steps](#deployment-steps)
- [Verification](#verification)
- [Project Structure](#project-structure)
- [Key Concepts](#key-concepts)
- [Cleanup](#cleanup)
- [References](#references)

---

## Architecture

\```
Internet
    │
    ▼
AWS Application Load Balancer (ALB)
    │  (provisioned via AWS Load Balancer Controller + Kubernetes Ingress)
    ▼
Kubernetes Ingress  ──  ingress-2048  (class: alb)
    │
    ▼
Kubernetes Service  ──  service-2048  (NodePort)
    │
    ▼
Deployment: deployment-2048  (5 replicas)
    │
    ▼
AWS Fargate  (Serverless — no EC2 nodes to manage)
    │
    ▼
EKS Cluster: demo-cluster  (ap-south-1 / Mumbai)
\```

---

## Tech Stack

| Tool | Version |
|------|---------|
| AWS CLI | 2.32.16 |
| kubectl | v1.28.15 |
| eksctl | 0.224.0 |
| Helm | v3.20.0 |
| Kubernetes | 1.34 |
| AWS Load Balancer Controller | v2.11.0 |
| AWS Region | ap-south-1 (Mumbai) |
| Compute | AWS Fargate (Serverless) |

---

## Prerequisites

- An AWS account with IAM permissions for EKS, EC2, IAM, and CloudFormation
- AWS CLI installed and configured
- `kubectl`, `eksctl`, and `helm` installed on your machine

> All installation and configuration commands are available in the [`/commands`](./commands) folder.

---

## Deployment Steps

> All commands for each step are available in the [`/commands`](./commands) folder.

### 1. Install eksctl
Download and install the `eksctl` CLI binary for your platform.

### 2. Configure AWS CLI
Set up your AWS credentials and default region using `aws configure`.

### 3. Create the EKS Cluster with Fargate
Provision the EKS control plane with a default Fargate profile using `eksctl`. This also installs core addons — `vpc-cni`, `kube-proxy`, `coredns`, and `metrics-server`.

### 4. Update kubeconfig
Merge the cluster credentials into your local kubeconfig so `kubectl` can communicate with the cluster.

### 5. Create a Fargate Profile for the App Namespace
Create a dedicated Fargate profile for the `game-2048` namespace so application pods are scheduled onto Fargate.

### 6. Deploy the 2048 Application
Apply the upstream manifest which creates the `game-2048` namespace, a 5-replica Deployment, a Service, and an Ingress resource.

### 7. Associate IAM OIDC Provider
Enable OIDC on the cluster so Kubernetes service accounts can assume IAM roles — required for the Load Balancer Controller.

### 8. Create IAM Policy for the Load Balancer Controller
Download the official IAM policy document and create the policy in your AWS account.

### 9. Create an IAM Service Account
Create a Kubernetes service account in the `kube-system` namespace linked to the IAM role via IRSA (IAM Roles for Service Accounts).

### 10. Install the AWS Load Balancer Controller via Helm
Add the EKS Helm chart repository and install the controller into the `kube-system` namespace.

### 11. Verify the Deployment
Confirm the controller pods are running and that the Ingress has been assigned an ALB DNS address.

---

## Verification

Once the ALB is fully provisioned (typically 2–3 minutes after Step 11), retrieve the Ingress address and open it in your browser to access the live 2048 game.

\```
NAME           CLASS   HOSTS   ADDRESS                                                                   PORTS
ingress-2048   alb     *       k8s-game2048-ingress2-xxxxxxxx.ap-south-1.elb.amazonaws.com               80
\```

---

## Project Structure

\```
.
├── README.md
├── commands/
│   ├── 01-install-eksctl.sh
│   ├── 02-configure-aws.sh
│   ├── 03-create-cluster.sh
│   ├── 04-update-kubeconfig.sh
│   ├── 05-create-fargate-profile.sh
│   ├── 06-deploy-app.sh
│   ├── 07-associate-oidc.sh
│   ├── 08-create-iam-policy.sh
│   ├── 09-create-service-account.sh
│   ├── 10-install-alb-controller.sh
│   └── 11-verify.sh
└── iam_policy.json
\```

---

## Key Concepts

**AWS Fargate** — Serverless compute for containers. Pods run without provisioning or managing EC2 nodes, and you pay only for the vCPU and memory your pods use.

**Fargate Profile** — Determines which pods (matched by namespace or labels) are scheduled onto Fargate. Separate profiles are used for system components and the application.

**AWS Load Balancer Controller** — A Kubernetes controller that watches `Ingress` resources and automatically provisions AWS ALBs. It replaces the legacy in-tree AWS cloud provider for load balancers.

**OIDC Provider** — Enables IAM Roles for Service Accounts (IRSA), allowing Kubernetes workloads to assume IAM roles securely without embedding credentials.

**eksctl** — An official CLI tool for creating and managing EKS clusters, backed by AWS CloudFormation under the hood.

---

## Cleanup

To avoid ongoing AWS charges, delete all resources when you are done. Refer to the [`/commands`](./commands) folder for the full cleanup commands.

This removes the EKS cluster, all Fargate profiles, CloudFormation stacks, and the associated IAM service account role. Remember to also delete the IAM policy created in Step 8 if it is no longer needed.

---

## References

- [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/)
- [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/)
- [eksctl Documentation](https://eksctl.io/)
- [Helm Documentation](https://helm.sh/docs/)
- [2048 Game Manifest (kubernetes-sigs)](https://github.com/kubernetes-sigs/aws-load-balancer-controller/tree/main/docs/examples/2048)
