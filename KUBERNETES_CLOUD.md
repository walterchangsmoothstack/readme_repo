# KUBERNETES CLOUD EKS READ ME

### Installation

Use the ```eksctl``` community package to create EKS cluster.  
  
> choco install -y eksctl  
> choco upgrade -y eksctl  

## Using the AWS GUI  
1) create an IAM Role eksClusterRole: SELECT EC2 + EKS Cluster => ```AmazonEKSClusterPolicy```  
2) create an IAM Role eksWorkerNodeRole: SELECT EC2 => ``` AmazonEKS_CNI_Policy``` ``` AmazonEKSWorkerNodePolicy``` ```AmazonEC2ContainerRegistryReadOnly```

## Using eksctl  
*Create a CloudFormation Stack that provisions everything required to run kubernetes resources in a cloud cluster.*
> eksctl create cluster --name myeks-cluster --version 1.21 --region us-west-1 --nodegroup-name linux-nodes --node-type t2.micro --nodes 2  

*Delete the cluster*
> ekstctl delete cluster --name myeks-cluster  

*Switch to EKS Cluster*
> aws eks --region us-west-1 update-kubeconfig --name utopia-cluster  

*Switch namespaces in Kubernetes*
> kubectl config set-context --current --namespace=nginx-ingress  
