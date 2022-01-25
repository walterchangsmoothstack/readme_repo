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
> eksctl create cluster --name myeks-cluster --version 1.21 --region us-west-1 --nodegroup-name linux-nodes --node-type t2.micro --nodes 2  --vpc-public-subnets=<public_subnet1,public_subnet2> --vpc-private-subnets=<private_subnet1,private_subnet2>

*Delete the cluster*
> ekstctl delete cluster --name myeks-cluster  

*Switch to EKS Cluster*
> aws eks --region us-west-2 update-kubeconfig --name wc  

*Switch namespaces in Kubernetes*
> kubectl config set-context --current --namespace=nginx-ingress  

## Expose Service to the outside

*Reference:* https://aws.amazon.com/premiumsupport/knowledge-center/eks-access-kubernetes-services/

*Requirements:*  
- Load Balancer service that acts as a gateway
- Ingress Controller (pod) that is linked to the load balancer
- Ingress Class for the Ingress Controller to function
- Ingress: which routes the traffic to other services
- ClusterRole & RBAC: which provides authorization to access resources in the Cluster
- ConfigMap with proxy-protocol, real-ip-header, set-real-ip-from

- TSL Secret: which is a secret provided for the default secret in the Ingress Controller

*Personal Notes:*
- Host is required for the Ingress rules. Try using wildcard if no DNS is set up ex. "\*.com"
- Unsure about where to configure port routing or if all applications for the ingress rule must have the same route. Expose port 80 on all applications for now
- Number of nodes must be sufficient for the number of pods running. https://github.com/awslabs/amazon-eks-ami/blob/master/files/eni-max-pods.txt


## Using Fargate and existing subnets

## [Using the Kubernetes Secrets Store CSI Driver](https://docs.aws.amazon.com/secretsmanager/latest/userguide/integrating_csi_driver.html)
