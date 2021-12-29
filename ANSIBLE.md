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


