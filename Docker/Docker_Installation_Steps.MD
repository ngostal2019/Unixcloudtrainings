# Installing Docker on Amazon Linux server

### Pre-requisites
1. Amazon Linux EC2 Instance

## Installation Steps

1. Create a user called ansadmin (on Control node and Managed host)  
   ```sh
   sudo -i
   useradd -m -d /home/ansadmin -c "Docker user " ansadmin
   echo "test123" | passwd --stdin ansadmin
   ```
1. Below command grant sudo access to ansadmin user. But we strongly recommended using "visudo" command if you are aware vi or nano editor.  (on Control node and Managed host)
   ```sh
   echo "ansadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
   exit
   su - ansadmin
   ```

1. Install docker and start docker services
   ```sh 
   sudo yum install docker -y 
   
   # start docker services
   systemctl start docker
   systemctl status docker

   # add user to docker group 
   sudo usermod -aG docker ansadmin
   docker --version
   ```

1. Enable password authentication to the docker server for the Ansible machine to connect to it.
   ```sh
   sudo vim /etc/ssh/sshd_config --> type /PasswordAuthentication --> hit `Enter` --> change it from `no` to `yes`
   sudo systemctl reload sshd
   ```

### Validation test
1. Create a tomcat docker container by pulling a docker image from the public docker registry
   ```sh
   docker run -d --name test-tomcat-server -p 8090:8080 tomcat:latest
   ```

## Docker Installation on CentOS server
##### Referance URL : https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-repository
### Pre-requisites

Please follow below steps to install docker CE on CentoOS server instance. For RedHat only Docker EE available 

1. Install the required packages.

   ```sh 
   sudo yum install -y yum-utils \
   device-mapper-persistent-data \
   lvm2
   ```
  
1. Use the following command to set up the stable repository.
 
   ```sh 
   sudo yum-config-manager \
   --add-repo \
   https://download.docker.com/linux/centos/docker-ce.repo
   ```

### INSTALLING DOCKER CE

1. Install the latest version of Docker CE.
   ```sh 
   sudo yum install docker-ce
   ```

   Note: If prompted to accept the GPG key, verify that the fingerprint matches 
060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35, and if so, accept it.

1. Start Docker.
   ```sh 
   sudo systemctl start docker
   ```

1. Verify that docker is installed correctly by running the hello-world image.
   ```sh
   sudo docker run hello-world
   ```
