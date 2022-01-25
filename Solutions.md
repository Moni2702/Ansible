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
# Implementations in file 3-provision-jenkins-ec2.yaml & 3-install-jenkins-ec2.yaml 

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
<summary>Exercise 5: xxxxxx </summary>
 <br />

**steps:**
```sh


```

</details>

******
