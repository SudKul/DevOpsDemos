## Simple DevOps Project -3 



### 1. Launch an EC2 instance for Docker host

### 2. Install docker on EC2 instance 
```sh 
# update the existing packages
sudo yum update
sudo  yum install docker
```

### 3. Create a new user for Docker management, and add that user to Docker (default) group. 
```sh
sudo useradd host2admin
sudo passwd host2admin
# Add the host2admin user to the "docker" user group 
sudo usermod -aG docker host2admin
# Add the $USER user to the "docker" user group. The current $USER is ec2-user
sudo usermod -a -G docker $USER
sudo reboot
```
### Start services 
```bash
# Reconnect using SSH. The public IP will cahnge after reboot
sudo service docker start
```


##### Optional
You would need to log out and log back in using SSH so that your group membership is re-evaluated or type the following command:
```
su -s dockeradmin
```
Verify that you can run docker commands without sudo.
```
docker run hello-world
```


### 4. Write a Docker file under /opt/docker

```bash
sudo su -
cd /opt
mkdir docker
cd docker
vi Dockerfile
```

```sh
# Pull base image 
From tomcat:8-jre8 

# Maintainer
MAINTAINER "Udacity" 

# copy war file on to container 
COPY ./webapp.war /usr/local/tomcat/webapps
```

Enable the password based authentication
```
vi /etc/ssh/sshd_config
# Comment the passwordauthentication line
 sudo service sshd restart
 ```
Change ownership permissions. Allow the new user to write here
```bash
 chown -R host2admin:host2admin /opt/docker/
 sudo service docker restart
```

### 5. Login to Jenkins console and add Docker server to execute commands from Jenkins  
* Manage Jenkins --> Manager plugins --> Install "Publish over SSH" plugin
* Manage Jenkins --> Configure system -->  Publish over SSH --> add Docker server and credentials

### 6. Create Jenkins job 
Create mySecondJob (Type: Maven project)

1. **Source Code Management**
 Repository : https://github.com/ValaxyTech/hello-world.git  
 Branches to build : */master  

2. **Build**
 Root POM: pom.xml  
 Goals and options : clean install package  
 
3. **Post Steps**
Add post-build steps: Choose "Send files or execute commands over SSH"
 Name: Host_2 
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

#### 7. Login to Docker host and check images and containers. (no images and containers)

### 8. Execute Jenkins job

### 9. Check images and containers again on Docker host. 
This time an image, **demo_image** and a container, **demo_container** will get created through Jenkins job

### 10. Access web application from browser which is running on container
```
<docker_host_Public_IP>:8888
```
