</details>

******

<details>
<summary>Exercise 1: Build & Deploy Java Artifact </summary>
 <br />

**preparation**
```sh
- Create an Ubuntu server (Ubuntu version 20.04) on any platform you want
- Configure ssh access to your server (open port 22, create ssh key access)
- Set "server IP address", "user" and "ssh private key path" inside "hosts-deploy-app" file
- Fix the test in the application (if you have not already done it in the Build tools module exercises) to allow building it successfully
Fix: In file - src/test/java/AppTest.java, line - 22, remove quotes from true to make it boolean.

- Install the "acl" package that includes "setfacl" command on the ubuntu machine, so that Ansible can set temporary file permissions correctly, when connecting to the server as an unprivileged user (ubuntu) and becoming another unprivileged user (my-user)
sudo apt-get update -y
sudo apt-get install -y acl
Link: https://docs.ansible.com/ansible/latest/user_guide/become.html#risks-of-becoming-an-unprivileged-user
```

**code**

```sh
# Implementation in file 1-build-and-deploy.yaml 

# execute with hosts-deploy-app host file
ansible-playbook -i hosts-deploy-app 1-build-and-deploy.yaml --extra-vars "linux_user=your-name project_dir=/absolute/path/to/java/gradle/project jar_name=bootcamp-java-project-1.0-SNAPSHOT.jar"
```

</details>

******

<details>
<summary>Exercise 2: Push Java Artifact to Nexus </summary>
 <br />

**code**
```sh
# Make sure you have a Nexus repository manager up and running with maven-snapshots repository

# Implementation in file 2-push-to-nexus.yaml 

# Execute 
ansible-playbook 2-push-to-nexus.yaml --extra-vars "nexus_url=http://nexus-ip:nexus-port nexus_user=admin nexus_password=admin-pass repository_name=maven-snapshots artifact_name=bootcamp-java-project artifact_version=1.0-SNAPSHOT jar_file_path=/absolute/path/to/jar/file/bootcamp-java-project-1.0-SNAPSHOT"

# NOTE: you can specify any inventory file (ansible-playbook -i hosts-deploy-app 2-push-to-nexus.yaml) to avoid the warning, even though we are using localhost, so we don't need an inventory file
```

</details>

******

<details>
<summary>Exercise 3: Install Jenkins on EC2 </summary>
 <br />

**steps**
```sh
# Implementations in files: 
- 3-provision-jenkins-ec2.yaml 
- 3-install-jenkins-ec2.yaml 

# Execute
ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/path/to/ssh-key/file aws_region=your-aws-region key_name=your-key-pair-name subnet_id=your-subnet-id ami_id=image-id-for-amazon-linux ssh_user=ec2-user" 

# NOTE: Optionally you can try getting some of these parameter values programatically using Ansible itself. You can also put them into a variables file instead of passing on cli. Or parameterise even more values inside the playbook, like version numbers of different tools etc. As you see you are very flexible in what you can do with Ansible.

ansible-playbook -i hosts-jenkins-server 3-install-jenkins-ec2.yaml --extra-vars "aws_region=your-aws-region"
```

</details>

******

<details>
<summary>Exercise 4: Install Jenkins on Ubuntu </summary>
 <br />

**steps**
```sh
# Implementations in files: 
- 3-provision-jenkins-ec2.yaml 
- 4-install-jenkins-ubuntu.yaml
- 4-host-amazon.yaml
- 4-host-ubuntu.yaml

# To Create and configure Jenkins on --ubuntu-- EC2 instance
ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/path/to/ssh-key/file aws_region=your-aws-region key_name=your-key-pair-name subnet_id=your-subnet-id ami_id=image-id-for-ubuntu ssh_user=ubuntu"

ansible-playbook -i hosts-jenkins-server 4-install-jenkins-ubuntu.yaml --extra-vars "host_os=ubuntu aws_region=your-aws-region"

# To Create and configure Jenkins on --amazon-linux-- EC2 instance
ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/path/to/ssh-key/file aws_region=your-aws-region key_name=your-key-pair-name subnet_id=your-subnet-id ami_id=image-id-for-ubuntu ssh_user=ubuntu"

ansible-playbook -i hosts-jenkins-server 4-install-jenkins-ubuntu.yaml --extra-vars "host_os=amazon-linux aws_region=your-aws-region"

# NOTES:
- When executing "3-provision-jenkins-ec2.yaml" script, the only difference between the 2 is the "ami_id" value. So you either provide the "amazon-linux" ami-id or "ubuntu" ami-id
- The shared tasks are all defined in the common "provision-jenkins-ec2.yaml" and the differenes are in respective "host-amazon" or "host-ubuntu" yaml files, which get dynamically selected, based on what "host_os" variable you provide

```

</details>

******

<details>
<summary>Exercise 5: Install Jenkins as a Docker Container </summary>
 <br />

**steps:**
```sh
# Implementation in files: 
- 3-provision-jenkins-ec2.yaml 
- 5-install-jenkins-docker.yaml

# Execute to provision the ubuntu Jenkins server
ansible-playbook 3-provision-jenkins-ec2.yaml --extra-vars "ssh_key_path=/path/to/ssh-key/file aws_region=your-aws-region key_name=your-key-pair-name subnet_id=your-subnet-id ami_id=image-id-for-ubuntu ssh_user=ubuntu"

# Execute to configure server to run jenkins as a docker container
ansible-playbook -i hosts-jenkins-server 5-install-jenkins-docker.yaml --extra-vars "aws_region=your-aws-region"

```

</details>

******

<details>
<summary>Exercise 6: Web server and Database server configuration </summary>
 <br />

**steps:**
```sh
# From local computer:
## Create an Ansible control (ubuntu) server in your AWS VPC 
xxx

# On Ansible control server:
## Install all the needed tools to execute the playbooks from there: 
### Install Ansible
sudo apt-get update
sudo apt-get install ansible

### Install Ansible role for mysql configuration
ansible-galaxy install geerlingguy.mysql

### Install pip3, boto3 and botocore
sudo apt install python3-pip
pip3 install boto3 botocore

## Configure aws credentials
### Create .aws folder in ubuntu user's home dir
mkdir .aws

### Copy your aws creds
vim .aws/credentials

## Configure ssh key for accessing database and web servers. You can copy the keys from your local computer to ansible control server
scp -i private-key-to-connect-to-ansible-control-server-from-localhost private-key-to-connect-to-database-web-servers-from-ansible-control-server ubuntu@ansible-server-ip:/home/ubuntu

## Set the correct aws region inside 6-inventory_aws_ec2.yaml file
regions: 
- your-aws-region

## Copy all your playbook files and configuration files to ansible server
scp -i private-key-to-connect-to-ansible-control-server-from-localhost dir-with-ansible-files ubuntu@ansible-server-ip:/home/ubuntu

## Provision both (ubuntu) servers inside the same VPC with ansible script 6-provision-servers.yaml
ansible-playbook 6-provision-servers.yaml --extra-vars "ssh_key_path=~/Downloads/boto3-server-key.pem aws_region=eu-west-3 key_name=boto3-server-key subnet_id=subnet-d74639be ami_id=ami-031eb8d942193d84f ssh_user=ubuntu"

# Comment in lines 6,7,8 inside ansible.cfg file, to enable the aws_ec2 plugin and configure remote user
enable_plugins = amazon.aws.aws_ec2
remote_user = ubuntu
private_key_file = private-key-to-connect-to-database-web-servers-from-ansible-control-server

# You can test the inventory file results with command and see the dynamic group names for referencing database and web servers:
ansible-inventory -i 6-inventory_aws_ec2.yaml --graph

# Configure both servers with ansible script 6-configure-servers.yaml using the dynamic inventory file
ansible-playbook -i 6-inventory_aws_ec2.yaml 6-configure-servers.yaml

# Make sure to open port 8080 for the web server and access the application via public-ip:8080 from browser to make sure the application was successfuly deployed :)

```

</details>

******
