# Integrating Kubernetes cluster with Ansible

1. Login to ansible server and copy public key onto kubernetes cluster master account 

1. Update hosts file with new group called k8s-bootstrap-server and its IP address. 

1. Create ansible playbooks to create **[deployment](https://github.com/ngostal2019/Unixcloudtrainings/blob/main/Kubernetes/k8s-uct-deployment.yml)** and **[services](https://github.com/ngostal2019/Unixcloudtrainings/blob/main/Kubernetes/k8s-uct-service.yml)** 
		
1.  Check for pods, deployments and services on kubernetes master
    ```sh 
    kubectl get pods -o wide 
    kubectl get deploy -o wide
    kubectl get service -o wide
    ```
	
1. Access application suing service IP
   ```sh
   wget <kubernetes-Master-IP>:31200
   ```
