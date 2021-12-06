# KUBERNETES CLOUD EKS READ ME

Use the ```eksctl``` community package to create EKS cluster.  
  
> choco install -y eksctl  
> choco upgrade -y eksctl  

Create a CloudFormation Stack that provisions everything required to run kubernetes resources in a cloud cluster.
> eksctl create cluster

### Using the AWS GUI  
1) create an IAM Role eksClusterRole: SELECT EC2 + EKS Cluster => ```AmazonEKSClusterPolicy```  
2) create an IAM Role eksWorkerNodeRole: SELECT EC2 => ``` AmazonEKS_CNI_Policy``` ``` AmazonEKSWorkerNodePolicy``` ```AmazonEC2ContainerRegistryReadOnly```

