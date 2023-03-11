# Deploy on a docker container using Ansible
# *Jenkins Job name:* `Deploy_on_Container_using_ansible`

### Pre-requisites

1. Jenkins server 
1. Docker-host server 
1. Ansible server
1. `Dockerfile` under *`/opt/docker`* on Ansible server
   ```sh 
   # Pull tomcat latest image from dockerhub 
   From ngostal/rockylinux9:tomcat10
   # Maintainer
   LABEL ORGANIZATION="UNIXCLOUDTRAININGS" 

   # copy war file on to container 
   COPY ./webapp.war /usr/local/tomcat/webapps
1. Create `create-car-brand-choice-image.yml` unser *`/opt/docker`* on Ansible server 
   ```sh
   ---
   - hosts: all
     #ansadmin doesn't need root access to create an image
     become: true 

     tasks:
     - name: building docker image
       command: "docker build -t car-brand-choice:v1.0.0 ." 
       args:
         chdir: /opt/docker
   ```
1. Create `create-car-brand-choice-image.yml` under *`/opt/docker`* on Ansible server
   ```ssh
   ---
   - hosts: all
     become: true

     tasks:
     - name: creating docker image using docker command
       command: docker run -d --name car-brand-choice-container -p 8080:8080 car-brand-choice:v1.0.0
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
      - Repository: `https://github.com/ngostal2019/Dev_code.git`
      - Branches to build : `*/main`  
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
                ansible-playbook -i /etc/ansible/hosts /opt/ansible/create-car-brand-choice-image.yml;
                ```

Save and run the job now.
