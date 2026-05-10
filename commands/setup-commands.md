# EKS Cluster Creation

eksctl create cluster --name demo-cluster --region ap-south-1 --fargate

# Update kubeconfig

aws eks update-kubeconfig --region ap-south-1 --name demo-cluster

# Create Fargate Profile

eksctl create fargateprofile \
--cluster demo-cluster \
--region ap-south-1 \
--name alb-sample-app \
--namespace game-2048

# Deploy Application

kubectl apply -f manifests/2048_full.yaml
