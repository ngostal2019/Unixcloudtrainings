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

Enable connection between  Docker and Jenkins
- `Manage Jenkins` > `Configure System` > `Publish Over SSH` > `SSH Servers` 

	- SSH Servers:
    - Name: `docker-server`
		- Hostname:`<ServerIP>`
		- username: `ansadmin`
               
       -  `Advanced` > chose `Use password authentication, or use a different key`
		 - password: `*******`
---------------------------------------------------
### Steps to create "deploy_to_docker_CI" Jenkin job
#### From Jenkins home page select "New Item"
   - Enter an item name: `deploy_to_docker_CI`
     - Description: `Deploy war file to Docker server and trigger downstream job invoke_ansible_commands_CD`
     
   - *Source Code Management:*
      - Repository: `https://github.com/ngostal2019/Dev_code.git`
      - Branches to build : `*/main`  
   - Check *GitHub hook trigger for GITScm polling*

   - *Build:*
     - Root POM: `pom.xml`
     - Goals and options: `clean install package`

 - *Post-build Actions*
    - Choose `Build other jobs`
      - *Projects to build*
        - Give the name of your Downstream job: `invoke_ansible_commands_CD`

 - *Post-build Actions*
   - Send build artifacts over SSH
     - *SSH Publishers*
      - SSH Server Name: `docker-server`
       - `Transfers` >  `Transfer set`
           - Source files: `webapp/target/*.war`
	       - Remove prefix: `webapp/target`
	       - Remote directory: `//opt//docker`

- Save

----------------------------------------------------------
### Steps to create "invoke_ansible_commands_CD" Jenkin job
#### From Jenkins home page select "New Item"
- Enter an item name: `invoke_ansible_commands_CD`
- Free Style Job
     - Description: `Invoke ansible command to build docker image on the docker server and push image to docker hub`
     - *Post-build Actions*
       - Send build artifacts over SSH
         - SSH Publishers
           - SSH Server: `ansible-server`
           - Exec command:
             ```sh
             /home/ansadmin/.local/bin/ansible-playbook -i /opt/ansible/hosts /opt/ansible/create-docker-image.yml
             ```
- Save
---------------------------------------------------------

Enable connection between Ansible-control-node and Jenkins and Kubernetes bootstrap server

### Steps to create "Deploy_on_Kubernetes_CD" Jenkin job
#### From Jenkins home page select "New Item"
   - Enter an item name: `Deploy_to_Kubernetes_CD`
   - Freestyle Project 
     - Description: `Invoke ansible command to build trigger kubernetes command to run manifest file on Kubernetes`
     - *Post-build Actions*  
        - Send build artifacts over SSH  
        - *SSH Publishers*  
            - SSH Server: `ansible-server`
            - Exec command: 
              ```sh 
              /home/ansadmin/.local/bin/ansible-playbook -i /opt/ansible/hosts /opt/kubernetes/play-k8s-uct-deploy.yml;
              /home/ansadmin/.local/bin/ansible-playbook -i /opt/ansible/hosts /opt/kubernetes/play-k8s-uct-service.yml;
              ```
- Save
- Push a new commit to test everything.
