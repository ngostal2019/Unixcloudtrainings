# Ansible Installation

Ansible is an open-source automation platform. It is very, very simple to set up and yet powerful. Ansible can help you with configuration management, application deployment, task automation.

### Pre-requisites

1. An AWS EC2 instance (on Control node)

### Installation steps:
#### on Amazon EC2 instance

1. Create a user called ansadmin (on Control node and Managed host)  
   ```sh
   sudo -i
   useradd -m -d /home/ansadmin -c "Ansible user controller" ansadmin
   echo "test123" | passwd --stdin ansadmin
   ```
1. Below command grant sudo access to ansadmin user. But we strongly recommended using "visudo" command if you are aware vi or nano editor.  (on Control node and Managed host)
   ```sh
   echo "ansadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
   exit
   su - ansadmin
   ```

1. Install python and python-pip
   ```sh
   sudo yum install python3 -y
   sudo yum install python3-pip -y
   ```

1. Install ansible using pip check for version
    ```sh
    pip3 install ansible
    ansible --version
   ```
   
1. Log in as a ansadmin user on master and generate ssh key (on Control node)
   ```sh 
   ssh-keygen
   ```
1. Copy keys onto all ansible managed hosts (on Control node)
   ```sh 
   ssh-copy-id ansadmin@<target-server>
   ```

1. Ansible server is in this case used to create images and store on docker registry. Hence install docker, start docker services and add ansadmin to the docker group. 
   ```sh
   sudo yum install docker -y
   
   # Start and enable docker services 
   sudo systemctl start docker
   sudo systemctl enable docker 
   
   # add user to docker group 
   sudo usermod -aG docker ansadmin
   ```
1. Create a directory /etc/ansible and create an inventory file called "hosts" add control node and managed hosts IP addresses to it.
   ```sh
   sudo mkdir /etc/ansible && sudo touch /etc/ansible/hosts
   ```
 
### Validation test
  
1. Run ansible command as ansadmin user it should be successful (Master)
   ```sh 
   ansible all -m ping
   ```
