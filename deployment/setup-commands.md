# EKS Cluster Creation and Application Deployment

## Create EKS Cluster

```bash
eksctl create cluster --name demo-cluster --region ap-south-1 --fargate
```

## Update kubeconfig

```bash
aws eks update-kubeconfig --region ap-south-1 --name demo-cluster
```

## Create Fargate Profile

```bash
eksctl create fargateprofile \
--cluster demo-cluster \
--region ap-south-1 \
--name alb-sample-app \
--namespace game-2048
```

## Deploy Application

```bash
kubectl apply -f manifests/2048_full.yaml
```

## Application Screenshot

![2048 App](./images/2048-app.png)
