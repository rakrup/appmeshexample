# Setup AWS Account 
aws configure   (Provide Access Key, Secret Access Key and Region)

# Create EKS cluster
eksctl create cluster --name mesh-test-cluster --version 1.21 --region ap-south-1 --zones ap-south-1a,ap-south-1b --nodegroup-name cls-nodes --node-type t2.micro --nodes 2

# Update KubeConfig
aws eks --region ap-south-1 update-kubeconfig --name mesh-test-cluster


# Add Helm Repo
helm repo add eks https://aws.github.io/eks-charts

# App Mesh CRD installation
kubectl apply -k "https://github.com/aws/eks-charts/stable/appmesh-controller/crds?ref=master"

# Create ControlPlane namespace
kubectl create ns appmesh-system

# OIDC Connect for the cluster
eksctl utils associate-iam-oidc-provider --region=ap-south-1 --cluster mesh-test-cluster --approve 

# IAM Role creation for app-mesh controller
eksctl create iamserviceaccount \
    --cluster mesh-test-cluster \
    --namespace appmesh-system \
    --name appmesh-controller \
    --attach-policy-arn  arn:aws:iam::aws:policy/AWSCloudMapFullAccess,arn:aws:iam::aws:policy/AWSAppMeshFullAccess \
    --override-existing-serviceaccounts \
    --approve

# Deploying App Mesh controller

helm upgrade -i appmesh-controller eks/appmesh-controller \
    --namespace appmesh-system \
    --set region=ap-south-1 \
    --set serviceAccount.create=false \
    --set serviceAccount.name=appmesh-controller