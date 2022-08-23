# Create a Mesh

kubectl apply -f mesh.yaml

# create a Namespace

kubectl apply -f ns.yaml

# Create VirtualNode

kubectl apply -f virtualNode.yaml

# Create Virtual Router (& Routes)

kubectl apply -f virtualRouter.yaml

# Create Virtual Service 

kubectl apply -f virtualService.yaml

# Create IAM Policy for the app envoy Access

aws iam create-policy --policy-name podinfo-policy --policy-document file://podInfoPolicy.json


# Create Service Account

eksctl create iamserviceaccount --region ap-south-1\
    --cluster mesh-test-cluster \
    --namespace mesh-workload \
    --name podinfo-sa \
    --attach-policy-arn  arn:aws:iam::903108665405:policy/podinfo-policy \
    --override-existing-serviceaccounts \
    --approve

# Deploy Application & Service

kubectl apply -n mesh-workload -f podinfoDeployment.yaml
kubectl apply -n mesh-workload -f podinfoService.yaml
