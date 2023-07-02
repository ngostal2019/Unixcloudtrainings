# Setup Kubernetes (K8s) Cluster on AWS with KOPS

* Before you attach the Ec2-k8s-bootstrap-role to your bootstrap server try to see if you can operate on aws from the command line with the following command on your ec2 instance
   ```sh
      aws s3 ls
      Error message will be: Unable to locate credentials. You can configure credentials by running "aws configure".
   ```
## 1st setup from AWS Cloud
1. Create an IAM role named `Ec2-k8s-bootstrap-role`  with `AmazonRoute53FullAccess`, `AmazonEC2FullAccess`, `IAMFullAccess`, `AmazonVPCFullAccess`, `AmazonSQSFullAccess`, `AmazonEventBridgeFullAccess` and `AmazonS3FullAccess` full access OR simply give `AdministratorAccess` (Not recommanded for production usage)

1. From the Ec2 instances page, attach the above IAM role to Amazon Linux EC2 instance:
   ```sh
      Actions --> Security --> Modify IAM role --> Click on the dropdown menu to choose the newly create IAM role
   ```

1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a purchased domain) OR you may skip this and make sure you create your cluster with ending name like `.k8s.local` [gossip-hostname](https://kops.sigs.k8s.io/gossip/) and you may have to use that name in the rest of the setups. We will create a `Route53 DNS hosted zone`
   ```sh
      Routeh53 --> hosted zones --> created hosted zone  
      Domain Name: uct.in
      Type: Private hosted zone for Amazon VPC
   ```

## 2nd setups on the EC2 VM
1. Create Amazon Linux EC2 instance `k8-bootstrap-server` in your region --> I will stick to `us-east-1` aws region which correspond to `North Virginia`

1. Create ansadmin username and provide admin privileges since we will use him/her with ansible machine and it will ease the process of manging the cluster
   ```sh
      Connect as the root user and type `sudo -i`
      Create the user --> `useradd -m -d /home/ansadmin -c "k8s boootstrap user ansadmin" ansadmin`
      Verify: `cat /etc/passwd`
      Set his/her password --> echo "test123" | passwd --stdin ansadmin
      Add him/her to the sudoers file --> `echo "ansadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers`
      Verify the entry in the sudoers file --> tail /etc/sudoers
      Type `exit`
      Switch to the user ansadmin --> `su - ansadmin`
   ```

1. Create ssh keys before creating cluster to acces the cluster for maintenace and other related activities
   ```sh
      ssh-keygen
   ```
1. Enable password authentication to the bootstrap server for the ansible machine to connect to it once the installation of the cluster is done
   ```sh
      sudo vim /etc/ssh/sshd_config --> type /PasswordAuthentication --> hit `Enter` --> change it from `no` to `yes`
      Type `sudo systemctl reload sshd`
   ```

1. Install AWSCLI
   ```sh
      verify the current aws cli available: aws --version
      cd /opt
      sudo  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      sudo yum install unzip python3 -y # install unzip & python3 in case of necessity
      sudo unzip awscliv2.zip
      sudo ./aws/install
      ./aws/install -i /usr/local/aws-cli -b /usr/local/bin #Execute this line if you don't want to use sudo from the previous line
      aws --version
      sudo rm -rf awscliv2.zip
    ```

1. Install kubectl Amazon Linux EC2 instance
   ```sh
      sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
      sudo chmod +x ./kubectl
      sudo mv ./kubectl /usr/local/bin/kubectl
   ```

1. Install kops on Amazon Linux EC2 instance
   ```sh
      sudo curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
      sudo chmod +x kops-linux-amd64
      sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```

1. Verify the ansadmin user is able to type the kops and kubectl commands
   ```sh
      kops version
      kubectl version --short --client
   ```

1. Try once again to list aws s3 bucket, if no error message appear, then the role was successsfully attached to the EC2 bootsrap server and we are good to move on to the next step
    ```sh
      aws ec2 describe-availability-zones --region us-east-1 # Attach the created role Ec2-k8s-bootstrap-role if you didn't do it before

      *Output: 
      `{
      "AvailabilityZones": [
         {
               "State": "available",
               "OptInStatus": "opt-in-not-required",
               "Messages": [],
               "RegionName": "us-east-1",
               "ZoneName": "us-east-1a",
               "ZoneId": "use2-az1",
               "GroupName": "us-east-1",
               "NetworkBorderGroup": "us-east-1",
               "ZoneType": "availability-zone"
         },
         {
               "State": "available",
               "OptInStatus": "opt-in-not-required",
               "Messages": [],
               "RegionName": "us-east-1",
               "ZoneName": "us-east-1b",
               "ZoneId": "use2-az2",
               "GroupName": "us-east-1",
               "NetworkBorderGroup": "us-east-1",
               "ZoneType": "availability-zone"
         },
         {
               "State": "available",
               "OptInStatus": "opt-in-not-required",
               "Messages": [],
               "RegionName": "us-east-1",
               "ZoneName": "us-east-1c",
               "ZoneId": "use2-az3",
               "GroupName": "us-east-1",
               "NetworkBorderGroup": "us-east-1",
               "ZoneType": "availability-zone"
         }
      ]
      }`
   ```
1. Configure the AWS region and the output format.
   - Note: If you attached the role to the user, then you may not need to attach the role to the EC2 machine and run aws configure to set up the AWS ACCESS ID and the AWS SECRET ACCESS KEY (They kind of represent the username and password of the user for programatic access)
      ```sh
         aws configure
         AWS Access Key ID [None]: your-access-key-id
         AWS Secret Access Key [None]: your-secret-key
         Default region name [None]: any-region
         Default output format [None]: json
      ```
   - In my case, I attached the role to the EC2 machine already. So, I just need to set up the region and the output format and leave the AWS ACCESS ID and the AWS SECRET ACCESS KEY empty
      ```sh
         aws configure
         AWS Access Key ID [None]: your-access-key-id
         AWS Secret Access Key [None]: your-secret-key
         Default region name [None]: us-east-1
         Default output format [None]: json
      ```

1. Create an S3 bucket (A bucket should have a unique name all across the AWS account)
   * NOTE: 
   In order for ServiceAccounts(In K8S) to use external permissions (aka IAM Roles for ServiceAccounts), you also need a bucket for hosting the OIDC documents. While you can create a separate bucket for it, for the sake of the training we will use a non recommended approach by granting a public ACL to the same bucket name. 
   Again, it's recommended to have a separate bucket for these files.
   The ACL must be public so that the AWS STS service can access them.

   ```sh
      aws s3api create-bucket \
      --bucket useast1.k8s.uct.in \
      --region us-east-1 \
      --object-ownership BucketOwnerPreferred

      *Output:
      `{
      "Location": "/useast1.k8s.uct.in"
      }`
   ```

1. Change the bucket public accessibility from private to public
   ```sh
      aws s3api put-public-access-block \
      --bucket useast1.k8s.uct.in \
      --public-access-block-configuration BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false
   ```

1. Now let's make our bucket publicly accessible by changing the *ACL*
   ```sh
      aws s3api put-bucket-acl \
      --bucket useast1.k8s.uct.in \
      --acl public-read
   ```

1. It is extremely recommended to version your bucket so the configuration can be recovered in the event of disaster. So, let's do it with the following command:
   ```sh
      aws s3api put-bucket-versioning \
      --bucket useast1.k8s.uct.in \
      --versioning-configuration Status=Enabled
   ```

1. Verify the private DNS DNS domain name we created
   ```sh
      dig ns uct.in

      Output:
      `; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.amzn2.5.2 <<>> ns uct.in
      ;; global options: +cmd
      ;; Got answer:
      ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44991
      ;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

      ;; OPT PSEUDOSECTION:
      ; EDNS: version: 0, flags:; udp: 4096
      ;; QUESTION SECTION:
      ;uct.in.                                IN      NS

      ;; ANSWER SECTION:
      uct.in.                 240     IN      NS      ns-1024.awsdns-00.org.
      uct.in.                 240     IN      NS      ns-1536.awsdns-00.co.uk.
      uct.in.                 240     IN      NS      ns-0.awsdns-00.com.
      uct.in.                 240     IN      NS      ns-512.awsdns-00.net.

      ;; Query time: 0 msec
      ;; SERVER: 172.31.0.2#53(172.31.0.2)
      ;; WHEN: Sat Mar 04 20:05:54 UTC 2023
      ;; MSG SIZE  rcvd: 173`
   ```

1. For consistency and simplicity, expose `CLUSTER_NAME` and `KOPS_STATE_STORE` environment variables (Use the same name as the bucket you created): 
   ```sh
      Name of the cluster --> `echo "export CLUSTER_NAME=useast1.k8s.uct.in" >> ~/.bash_profile`
      KOPS s3 state store --> `echo "export KOPS_STATE_STORE=s3://useast1.k8s.uct.in" >> ~/.bash_profile`
      Verify it --> cat ~/.bash_profile
      Source the file --> source ~/.bash_profile
      Verify they were exported successfully --> echo $CLUSTER_NAME && echo $KOPS_STATE_STORE
   ```

1. Create kubernetes cluster definitions on S3 bucket
   ```sh
      kops create cluster --name=$CLUSTER_NAME \
      --cloud=aws \
      --zones=us-east-1a \
      --node-count=2 \
      --node-size=t2.micro \
      --ssh-public-key=~/.ssh/id_rsa.pub \
      --dns-zone=uct.in \
      --dns private
   ```

1. Suggestions from kops(optional):
 * list clusters with: kops get cluster
 * edit this cluster with: kops edit cluster useast1.k8s.uct.in
 * edit your node instance group: kops edit ig --name=useast1.k8s.uct.in nodes-us-east-1a
 * edit your master instance group: kops edit ig --name=useast1.k8s.uct.in master-us-east-1a

1. Create kubernetes cluser
    ```sh
      kops update cluster --name useast1.k8s.uct.in --yes --admin

      *Output:
         I0304 18:54:53.040440     757 executor.go:111] Tasks: 0 done / 93 total; 45 can run
         I0304 18:54:53.163658     757 keypair.go:225] Issuing new certificate: "etcd-clients-ca"
         I0304 18:54:53.176261     757 keypair.go:225] Issuing new certificate: "apiserver-aggregator-ca"
         W0304 18:54:53.198603     757 vfs_castore.go:382] CA private key was not found
         I0304 18:54:53.200602     757 keypair.go:225] Issuing new certificate: "etcd-manager-ca-main"
         I0304 18:54:53.317823     757 keypair.go:225] Issuing new certificate: "etcd-peers-ca-main"
         I0304 18:54:53.417233     757 keypair.go:225] Issuing new certificate: "etcd-peers-ca-events"
         I0304 18:54:53.437266     757 keypair.go:225] Issuing new certificate: "etcd-manager-ca-events"
         W0304 18:54:53.699842     757 vfs_castore.go:382] CA private key was not found
         I0304 18:54:53.819815     757 keypair.go:225] Issuing new certificate: "kubernetes-ca"
         I0304 18:54:53.907859     757 keypair.go:225] Issuing new certificate: "service-account"
         I0304 18:54:55.754526     757 executor.go:111] Tasks: 45 done / 93 total; 18 can run
         I0304 18:54:56.858773     757 executor.go:111] Tasks: 63 done / 93 total; 26 can run
         I0304 18:54:57.990075     757 executor.go:111] Tasks: 89 done / 93 total; 2 can run
         I0304 18:54:59.090945     757 executor.go:155] No progress made, sleeping before retrying 2 task(s)
         I0304 18:55:09.091337     757 executor.go:111] Tasks: 89 done / 93 total; 2 can run
         I0304 18:55:10.543683     757 executor.go:111] Tasks: 91 done / 93 total; 2 can run
         I0304 18:55:10.625358     757 executor.go:111] Tasks: 93 done / 93 total; 0 can run
         I0304 18:55:11.061880     757 dns.go:238] Pre-creating DNS records
         I0304 18:55:11.353764     757 update_cluster.go:326] Exporting kubeconfig for cluster
         W0304 18:55:11.353822     757 create_kubecfg.go:90] Did not find API endpoint for gossip hostname; may not be able to reach cluster
         kOps has set your kubectl context to useast1.k8s.uct.in

         Cluster is starting.  It should be ready in a few minutes.

         Suggestions:
         * validate cluster: kops validate cluster --wait 10m
         * list nodes: kubectl get nodes --show-labels
         * ssh to the master: ssh -i ~/.ssh/id_rsa ubuntu@api.useast1.k8s.uct.in
         * the ubuntu user is specific to Ubuntu. If not using Ubuntu please use the appropriate user based on your OS.
         * read about installing addons at: https://kops.sigs.k8s.io/addons.
   ```

1. Validate your cluster (Your cluster will be ready between 5 - 10 minutes)
     ```sh
         kops validate cluster --wait 10m

         * Note: It is ok to see a message that looks the same as the below one:
         `Validation Failed
            W0304 20:26:21.456495   32604 validate_cluster.go:232] (will retry): cluster not yet healthy
            INSTANCE GROUPS
            NAME                    ROLE    MACHINETYPE     MIN     MAX     SUBNETS
            master-us-east-1a       Master  t3.medium       1       1       us-east-1a
            nodes-us-east-1a        Node    t2.small        2       2       us-east-1a

            NODE STATUS
            NAME    ROLE    READY

            VALIDATION ERRORS
            KIND    NAME            MESSAGE
            dns     apiserver       Validation Failed

            The dns-controller Kubernetes deployment has not updated the Kubernetes cluster's API DNS entry to the correct IP address.  The API DNS IP address is the placeholder address that kops creates: 203.0.113.123.  Please wait about 5-10 minutes for a master to start, dns-controller to launch, and DNS to propagate.  The protokube container and dns-controller deployment logs may contain more diagnostic information.  Etcd and the API DNS entries must be updated for a kops Kubernetes cluster to start.`
         It is because the nodes have not joined the cluster yet.
    ```

1. After approximately 10 min, the ouput will change to below message
   ```sh
      INSTANCE GROUPS
      NAME                    ROLE    MACHINETYPE     MIN     MAX     SUBNETS
      master-us-east-1a       Master  t3.medium       1       1       us-east-1a
      nodes-us-east-1a        Node    t2.small        2       2       us-east-1a

      NODE STATUS
      NAME                    ROLE    READY
      i-06b1b329305559e3a     master  True
      i-0d8b62b09745a2228     node    True
      i-0dae890d8efb45732     node    True

      `Your cluster useast1.k8s.uct.in is ready`
   ``` 

1. To list nodes
   ```sh
      kubectl get nodes
   ```

## Deploying pods on Kubernetes on AWS
1. Testing a manual deployment of our application from the cluster, we can use the procedure below:
   ```sh
      kubectl create deployment sample-nginx --image=nginx --replicas=2 --port=80
      kubectl create deployment devops-tomcat --image=ngostal/devops-tomcat-image:v1 --replicas=2 --port=8080
      kubectl get deployments
      kubectl get replicaset
      kubectl get pods
   ```

1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them.
   ```sh
      kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
      kubectl expose deployment devops-tomcat --port=8080 --type=LoadBalancer
      kubectl get services -o wide
      * Output:
      NAME            TYPE           CLUSTER-IP      EXTERNAL-IP                                                             PORT(S)          AGE     SELECTOR
      devops-tomcat   LoadBalancer   100.65.19.109   `a2b2bd8ea93b147a58f64c4e46250a9c-52683090.us-east-1.elb.amazonaws.com`   8080:32438/TCP   5m52s   app=devops-tomcat
      kubernetes      ClusterIP      100.64.0.1      <none>
      * Note: You should wait for a while for the EC2 instances to be registered by the LoadBalancer after creating the service
   ```
1. Verify the Loadbalancer was created under the EC2 instances tab

1. Open the LoadBalancer port number on the aws security group and acces the app.
   ```sh
      http://a2b2bd8ea93b147a58f64c4e46250a9c-52683090.us-east-1.elb.amazonaws.com:8080/webapp/
   ```

## Cleanups
1. To delete cluster
   ```sh
      kops delete cluster --name=$CLUSTER_NAME --yes
   ```

1. Delete the *Route53* and *S3 Bucket*, *EC2 Bootstrap VM*, *IAM roles* created earlier

## Installation reference
1. The above installation procedure have been simplified and taken from the KOPS website available [here](https://kops.sigs.k8s.io/getting_started/aws/) 