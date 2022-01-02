# ANSIBLE

```
pip install ansible
pip install boto
pip install boto3
```
## Storing AWS credentials
For local, I am storing the AWS credentials as global variables. If using an EC2 instance, then use an IAM role attached to the EC2 with ```AmazonEC2FullAccess```  
> export AWS_ACCESS_KEY=\<AWS Access Key\>  
> export AWS_SECRET_KEY=\<AWS Secret Key\>  
### Provide credentials in .yaml file:
> aws_access_key: "{{ lookup('env', 'AWS_ACCESS_KEY') }}"  
> aws_secret_key: "{{ lookup('env', 'AWS_SECRET_KEY') }}"  
### Create an EC2 Instance and add it to Host Group.
```
  tasks:
    - name: Creating EC2 instance
      ec2:
        instance_type: t2.micro
        key_name: "{{ keypair }}"
        image: "{{ image }}"
        region: "{{ region }}"
        aws_access_key: "{{ aws_access_key }}"
        aws_secret_key: "{{ aws_secret_key }}"
        vpc_subnet_id: "{{ vpc_subnet_id }}"
        group_id: "{{ security_group }}"
        assign_public_ip: yes
        count: 1
        wait: true
        instance_tags:
          Name: users_microservice
      register: ec2
      
    - name: Add new instance to host group
      add_host:
        hostname: "{{ item.public_ip }}"
        groupname: launched
      loop: "{{ ec2.instances }}"
```
### Configure EC2 Instances in Host Group
```
- name: Configure instance(s)
  hosts: launched
  become: yes
  tasks:
    - name: this task will install httpd and ensure apache is installed
      yum: name=httpd state=present
    - name: start apache service
      service: name=httpd enabled=yes state=started
    - name: pull an image
      docker_image:
        name: waltchang97/utopia-users-microservice
```

### Starting Instances

1) Start the instances using the ec2 module:
```
  - name: Start Users Instance
    ec2:
      region: "{{ region }}"
      instance_tags:
          Name: users_microservice
      state: running
    register: users
```
2) Wait for SSH to be available and then retrieve the instance's public IP address
```
  - name: Wait for Users SSH to come up
    delegate_to: "{{ item.public_dns_name }}"
    wait_for_connection:
      delay: 60
      timeout: 320
    loop: "{{ users.instances }}"
```
3) Add the instance to a host group *(Be careful not to add any terminated/stopped instances when filtering by tag. Use the instance-state-running attribute to add only desired instances)* and start the docker service and then the container. To avoid running a container that is already up use:
>       when: users_container.exists and users_container.container['State']['Running'] == false

### Notes for using Ansible locally
1) To install docker on the remote server:  
  - install docker ``` yum install docker -y``` 
  - install docker SDK for python  
  ```   
  - name: Install Docker Module for Python
    pip:
      name: docker
  ```
 2) If there are issues with pip, make sure that the correct python interpreter is being used. Set ```interpreter_python=/usr/bin/python3``` in .ansible.cfg. 
 3) If there are issues with docker, try using the community version. *Though this is not likely the problem if referencing the docker_container module instead of the community.docker.docker_container module*    
       > ansible-galaxy collection install community.docker  
 4) For secrets, use the amazon.aws.aws_secret module:  
    ```"{{ lookup('amazon.aws.aws_secret', 'utopia-secrets.SECRET_KEY', region=region, aws_access_key=aws_access_key, aws_secret_key=aws_secret_key, nested=true) }}"```  
  


