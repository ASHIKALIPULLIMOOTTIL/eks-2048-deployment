# ALB Ingress Setup on AWS EKS

## Check Application Pods

```bash
kubectl get pods -n game-2048
```

### Pod Lifecycle

Initially:

```text
Pending
```

Reason:
Fargate resources are being provisioned.

Then:

```text
ContainerCreating
```

Reason:
Container image is downloading and starting.

Finally:

```text
Running
```

Meaning:
Application pods are active and healthy.

Watch pods in real time:

```bash
kubectl get pods -n game-2048 -w
```

---

# Check Ingress

```bash
kubectl get ingress -n game-2048
```

Initially:

```text
ADDRESS empty
```

Reason:
AWS Load Balancer Controller is not installed yet.

---

# Enable OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster demo-cluster \
  --region ap-south-1 \
  --approve
```

### Why OIDC?

OIDC enables:

```text
Kubernetes Pods → Assume AWS IAM Roles
```

Required for:

- AWS Load Balancer Controller
- IAM Roles for Service Accounts (IRSA)

---

# Create IAM Policy

Download policy:

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```

Create policy:

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

The policy allows:

- ALB creation
- Target group management
- Security group modification

---

# Create IAM Service Account

```bash
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --region=ap-south-1 \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<ACCOUNT-ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

This creates:

- Kubernetes Service Account
- AWS IAM Role

Used by AWS Load Balancer Controller.

---

# Install Helm

Install Helm:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verify:

```bash
helm version
```

Add EKS repository:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

---

# Install AWS Load Balancer Controller

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=ap-south-1 \
  --set vpcId=<YOUR-VPC-ID>
```

### Purpose

The controller:

- Watches Ingress resources
- Creates AWS ALB automatically
- Routes traffic to Kubernetes services

---

# Verify Controller

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

Expected:

```text
2/2 running
```

Check system pods:

```bash
kubectl get pods -n kube-system
```

Expected:

- aws-load-balancer-controller
- coredns
- metrics-server

---

# Verify Ingress

```bash
kubectl get ingress -n game-2048
```

Expected:

```text
k8s-game2048-ingress....elb.amazonaws.com
```

This is the AWS Application Load Balancer DNS endpoint.

---

# Access Application

Open the ALB URL in browser:

```text
http://k8s-game2048-ingress....elb.amazonaws.com
```

The 2048 application should now be publicly accessible.

---

# Architecture

```text
Internet
   ↓
AWS ALB
   ↓
Ingress
   ↓
Service
   ↓
Pods
   ↓
AWS Fargate
```

---

# Cleanup

Delete cluster after usage:

```bash
eksctl delete cluster --name demo-cluster --region ap-south-1
```
