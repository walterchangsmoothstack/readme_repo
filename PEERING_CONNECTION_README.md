# PEERING CONNECTION READ ME

### GOAL: Allow connection to a private RDS in one VPC from a different VPC.


> 1) Create a VPC with CIDR Block 10.1.0.0/16 (VPC with RDS is 10.0.0.0/16)
> 2) Create two subnets with CIDR Blocks 10.1.0.0/24 and 10.1.1.0/24
> 3) Set enable auto-assign IPv4 and enable DNS hostnames on subnets and VPC respectively. DNS resolutions should be default enabled  
> 4) Create an Internet Gateway and attach it to your ecs VPC, and add it to that VPC's route table.  
> 5) Create a Peering Connection with requester and accepters as your two VPC's (order should not matter)  
> 6) Accept the Peering Connection Request.  
> 7) Configure your Route Tables -> Choose the route table in which your RDS' private subnets are associated with and add another route: 10.1.0.0/16 -- peering connection we created.  
> 8) Do the same with your other VPC's route table (I thought it would work without doing this step but apparently not) 10.0.0.0/16 -- peering connection we created.  
> 9) Configure security group of RDS to accept all TCP from the IP CIDR Block of the VPC (10.1.0.0/16)  


### Accomplish this without creating a new VPC:  
> 1) Create a peering connection between your default VPC and the RDS's VPC. 
> 2) Configure Route Tables to have routes pointing to the other VPC CIDR Block and the peering connection.
> 3) Configure security group of RDS to accept all TCP from the IP CIDR Block of the VPC (10.1.0.0/16)


