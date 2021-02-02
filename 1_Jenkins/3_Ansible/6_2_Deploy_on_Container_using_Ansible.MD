alias a="docker ps;docker images;ls -l;date"



# Deploy on a docker container using Ansible
# *Jenkins Job name:* `Deploy_on_Container_using_ansible`

### Pre-requisites

1. Jenkins server 
1. Docker-host server 
1. Ansible server
1. `Dockerfile` under *`/opt/docker`* on Ansible server **[Get Help Here]()**
   ```sh 
---->   
   # Pull tomcat latest image from dockerhub 
   From tomcat
   # Maintainer
   MAINTAINER "AR Shankar" 

   # copy war file on to container 
   COPY ./webapp.war /usr/local/tomcat/webapps
   
   
   
1. Create `create-docker-image.yml` unser *`/opt/docker`* on Ansible server **[Get Help Here]()**
   ```sh

[ansadmin@ip-172-31-92-7 docker]$ ls
Dockerfile  hosts  simple-devops-project.yaml  webapp.war


[ansadmin@ip-172-31-92-7 docker]$ cat simple-devops-project.yaml 
---
- hosts: all
  become: true 
  tasks:
  - name: building docker image
    command: "docker build -t simple-devops-image ." 
    args:
     chdir: /opt/docker

  - name: Create container using devops image
    command: docker run -d --name simple-devops-container -p 8080:8080 simple-devops-image


cat simple-devops-project_2.yaml 
---
- hosts: all
  become: true 
  tasks:
  - name: Stop current running container
    command: docker container stop simple-devops-container
    ignore_errors: yes
    
  - name: Remove stopped  Docker container
    command: docker rm simple-devops-container
    ignore_errors: yes

  - name: Remove Docker image
    command: docker rmi simple-devops-image
    ignore_errors: yes


  - name: building docker image
    command: "docker build -t simple-devops-image ." 
    args:
     chdir: /opt/docker

  - name: Create container using devops image
    command: docker run -d --name simple-devops-container -p 8080:8080 simple-devops-image
    
    

----------> Playbook-1--------  
#Create image  on Ansible system  via war file, tag it  and push to dockerhub  and remove local image

---
- hosts: all
  become: true

  tasks:
  - name: create docker image using war file
    command: docker build -t simple-devops-image:latest .
    args:
      chdir: /opt/docker

  - name: create tag to image
    command: docker tag simple-devops-image devopsharish/simple-devops-image

  - name: Docker login
    command: docker login -u="devopsharish" -p="1harish"

  - name: push image on to dockerhub
    command: docker push devopsharish/simple-devops-image

  - name: remove docker images form ansible server
    command: docker rmi simple-devops-image:latest devopsharish/simple-devops-image
    ignore_errors: yes





----------> Playbook-2--------  
#Run on all target env.---   Pull and create containers

- hosts: all
  become: true 
  tasks:
  - name: Stop current running container
    command: docker container stop simple-devops-container
    ignore_errors: yes
    
  - name: Remove stopped  Docker container
    command: docker rm simple-devops-container
    ignore_errors: yes

  - name: Remove Docker image
    command: docker rmi devopsharish/simple-devops-image:latest
    ignore_errors: yes

 # - name: building docker image
 #  command: "docker build -t simple-devops-image ." 
 #  args:
 #   chdir: /opt/docker
   
  - name: Pull Docker image from dockerhub
    command: docker pull devopsharish/simple-devops-image:latest
     

  - name: Create container using devops image
    command: docker run -d --name simple-devops-container -p 8080:8080 devopsharish/simple-devops-image:latest


   ```

### Integration between Ansible-control-node and Jenkins

Install "publish Over SSH"
 - `Manage Jenkins` > `Manage Plugins` > `Available` > `Publish over SSH`

Enable connection between Ansible-control-node and Jenkins

- `Manage Jenkins` > `Configure System` > `Publish Over SSH` > `SSH Servers` 

	- SSH Servers:
                - Name: `ansible-server`
		- Hostname:`<ServerIP>`
		- username: `ansadmin`
               
       -  `Advanced` > chose `Use password authentication, or use a different key`
		 - password: `*******`
 
### Steps to create "Deploy_on_Container_using_ansible" Jenkin job
#### From Jenkins home page select "New Item"
   - Enter an item name: `Deploy_on_Container_using_ansible`
     - Copy from: `Deploy_on_Container`
     
   - *Source Code Management:*
      - Repository: `https://github.com/yankils/hello-world.git`
      - Branches to build : `*/master`  
   - *Poll SCM* :      - `* * * *`

   - *Build:*
     - Root POM:`pom.xml`
     - Goals and options: `clean install package`

 - *Post-build Actions*
   - Send build artifacts over SSH
     - *SSH Publishers*
      - SSH Server Name: `ansible-server`
       - `Transfers` >  `Transfer set`
            - Source files: `webapp/target/*.war`
	       - Remove prefix: `webapp/target`
	       - Remote directory: `//opt//docker`
	       - Exec command: 
                ```sh 
                ansible-playbook -i /opt/docker/hosts /opt/docker/create-docker-image.yml;
                ```

Save and run the job now.
