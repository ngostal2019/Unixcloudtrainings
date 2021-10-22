# Setup Kubernetes (K8s) Cluster on AWS


1. Create Ubuntu EC2 instance
1. install AWSCLI
   ```sh
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    sudo apt install unzip python3 # install unzip & python3 in case of necessity
    unzip awscliv2.zip
    sudo ./aws/install
    ./aws/install -i /usr/local/aws-cli -b /usr/local/bin #Execute this line if you don't want to use sudo from the previous line
    ```

1. Install kubectl on ubuntu instance
   ```sh
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
   ```

1. Install kops on ubuntu instance
   ```sh
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```
1. Create an IAM user/role  with Route53, EC2, IAM, VPC and S3 full access OR simply give AdministratorAccess (Not recommanded for production usage)

1. Attach IAM role to ubuntu instance
   ```sh
   # Note: If you create IAM user with programmatic access then provide Access keys. Otherwise region information is enough
   aws configure
    ```

1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
   ```sh
   Routeh53 --> hosted zones --> created hosted zone  
   Domain Name: uct1.in
   Type: Private hosted zone for Amazon VPC
   ```

1. create an S3 bucket (A bucket should have a unique name all across the AWS account)
   ```sh
    aws s3 mb s3://devops1.k8s.uct1.in 
   ```
1. Expose environment variable (Use the same name as the bucket you created): 
   ```sh
    export KOPS_STATE_STORE=s3://devops1.k8s.uct1.in 
   ```

1. Create sshkeys before creating cluster
   ```sh
    ssh-keygen
   ```

1. Create kubernetes cluster definitions on S3 bucket
   ```sh
   kops create cluster --cloud=aws --zones=us-east-2a --name=devops1.k8s.uct1.in --node-count=2 --ssh-public-key=~/.ssh/id_rsa.pub --dns-zone=uct1.in --dns private
    ```
1. Suggestions from kops(optional):
 * list clusters with: kops get cluster
 * edit this cluster with: kops edit cluster devops1.k8s.uct1.in
 * edit your node instance group: kops edit ig --name=devops1.k8s.uct1.in nodes-us-east-2a
 * edit your master instance group: kops edit ig --name=devops1.k8s.uct1.in master-us-east-2a

1. Create kubernetes cluser
    ```sh
    kops update cluster devops1.k8s.uct1.in --yes
    ```

1. Validate your cluster (Your cluster will be ready between 5 - 10 minutes)
     ```sh
      kops validate cluster
    ```

1. To list nodes
   ```sh
   kubectl get nodes
   ```

1. To delete cluster
    ```sh
     kops delete cluster devops1.k8s.uct1.in --yes
    ```
   
#### Deploying Nginx pods on Kubernetes
1. Deploying Nginx Container
    ```sh
    kubectl create deployment sample-nginx --image=nginx --replicas=2 --port=80
    # kubectl create deployment devops-tomcat --image=ngostal/devops-tomcat-image:v1 --replicas=2 --port=8080
    kubectl get pods
    kubectl get deployments
   ```

1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them.
   ```sh
   kubectl expose deployment devops-tomcat --port=80 --type=LoadBalancer
   # kubectl expose deployment devops-tomcat --port=8080 --type=LoadBalancer
   kubectl get services -o wide
   ```
