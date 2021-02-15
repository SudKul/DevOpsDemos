## Simple DevOps Project -3 



#### 1. Launch a new host EC2 instance 
Let's name it as Host 2. On the Host 1, we already have Jenkins running insdie a container. 

#### 2. Install docker on EC2 instance 
Connect to the Host 2 using SSH, and install docker:
```sh 
# update the existing packages
sudo yum update
sudo  yum install docker
```

#### 3. Create a new user for Docker management, and add that user to Docker (default) group. 
```sh
sudo useradd host2admin
sudo passwd host2admin
# Add the host2admin user to the "docker" user group 
sudo usermod -aG docker host2admin
# Add the $USER user to the "docker" user group. The current $USER is ec2-user
sudo usermod -a -G docker $USER
sudo reboot
```

#### 4. Start services 
```bash
# Reconnect using SSH. The public IP will cahnge after reboot
sudo service docker start
# Verify that you can run docker commands without sudo.
docker run hello-world
```


#### 5. Write a Docker file under /opt/docker/ directory
```bash
sudo su -
cd /opt
mkdir docker
cd docker
vi Dockerfile
```
Add the following to the new Dockerfile
```sh
# Pull base image 
From tomcat:8-jre8 

# Maintainer
MAINTAINER "Udacity" 

# copy war file on to container 
COPY ./webapp.war /usr/local/tomcat/webapps
```
#### 6. Enable Jenkins' access to the Docker 
Jenkins will attempt to write files in the Docker as the newly created user "host2admin". Therefore, enable the password based authentication
```
vi /etc/ssh/sshd_config
# Comment the passwordauthentication line
sudo service sshd restart
```
Change ownership permissions, allowing the new user "host2admin" to write here
```bash
 chown -R host2admin:host2admin /opt/docker/
 sudo service docker restart
```

#### 7. Login to Jenkins console and add Docker server to execute commands from Jenkins  
* Manage Jenkins --> Manager plugins --> Install "Publish over SSH" plugin
* Manage Jenkins --> Configure system -->  Publish over SSH --> add Docker server and credentials

#### 8. Create Jenkins job 
Create mySecondJob (Type: Maven project)

1. **Source Code Management**
 Repository : https://github.com/ValaxyTech/hello-world.git  
 Branches to build : */master  

2. **Build**
 Root POM: pom.xml  
 Goals and options : clean install package  
 
3. **Post Steps**
Add post-build steps: Choose "Send files or execute commands over SSH"
 Name: Host_2 (Choose Verbose mode)
 Source files	: `webapp/target/*.war`
 Remove prefix	: `webapp/target`
 Remote directory	: `//opt//docker`  
 Exec command[s]	: 
```sh
docker stop demo_image;  docker rm -f demo_image; docker image rm -f demo_image; cd /opt/docker; docker build -t demo_image .
```
**Add a Transfer Set**<br>

  Exec command	: 
  ```
  docker run -d --name demo_container -p 8888:8080 demo_image
  ```  

#### 9. Login to Docker host and check images and containers. (no images and containers)

#### 10. Execute Jenkins job, and Check images and containers again on the Host 2. 
This time an image, **demo_image** and a container, **demo_container** will get created through Jenkins job

#### 10. Access web application from browser which is running on container
```
<docker_host_Public_IP>:8888
```
