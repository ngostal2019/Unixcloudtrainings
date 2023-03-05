## Integration Kubernetes with Jenkins

# *Jenkins CI Job:* `Deploy_Car-brand-choice_on_K8S`

### Pre-requisites

1. Jenkins server 
1. Ansible server
1. Kubernetes cluster
 
### Steps to create "Deploy_Car-brand-choice_on_K8S" Jenkins job
#### From Jenkins home page select "New Item"
   - Enter an item name: `Deploy_Car-brand-choice_on_K8S`
      - Choose Maven Project
     
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
                ansible-playbook -i /opt/docker/hosts /opt/docker/create-car-brand-choice-image.yml --limit localhost;
                ```

Save and run the job.

# *Jenkins CD Job:* `Deploy_on_Kubernetes-CD`

### Steps to create "Deploy_on_Kubernetes_CI" Jenkin job
#### From Jenkins home page select "New Item"
   - Enter an item name: `Deploy_on_Kubernetes_CI`
     - Freestyle Project
	 
  - *Post-build Actions*  
    - Send build artifacts over SSH  
      - *SSH Publishers*  
	       - Exec command: 
                ```sh 
                ansible-playbook -i /opt/docker/hosts /opt/docker/play-k8s-uct-deploy.yml;
                ansible-playbook -i /opt/docker/hosts /opt/docker/play-k8s-uct-service.yml;
                ```
Save and run the job.
