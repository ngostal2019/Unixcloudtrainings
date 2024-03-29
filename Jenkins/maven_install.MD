#  Install & configure Maven build tool on Jenkins
Maven is a code build tool which used to convert your code to an artifact. this is a widely used plugin to build in continuous integration


#### Prerequisites
1. Jenkins server

#### Install Maven on Jenkins
1. Download maven packages https://maven.apache.org/download.cgi onto Jenkins server. In this case, I am using /opt/maven as my installation directory
 - Link : https://maven.apache.org/download.cgi
    ```sh
     # Creating maven directory under /opt
     sudo mkdir /opt/maven
     cd /opt/maven
     # downloading maven version 3.6.3
     sudo wget https://dlcdn.apache.org/maven/maven-3/3.9.0/binaries/apache-maven-3.9.0-bin.tar.gz 
     sudo tar -xvzf apache-maven-3.9.0-bin.tar.gz
     sudo mv apache-maven-3.9.0-bin.tar.gz maven39
     sudo rm -rf apache-maven-3.9.0-bin.tar.gz
     ```
	
1. Setup M2_HOME and M2 paths in .bash_profile of the user and add these to the path variable
   ```sh
   vi ~/.bash_profile
   M2_HOME=/opt/maven/maven39
   PATH=<Existing_PATH>:$M2_HOME:$M2_HOME/bin
   ```
#### Checkpoint 
1. Run the command below to load your configuration 
   ```sh
   source .bash_profile
  ```
1. Check maven version
   mvn --version

   So far we have completed the installation of maven software to support maven plugin on the jenkins console. 
   Let's jump onto Jenkins to complete the remaining steps. 

# Setup maven on Jenkins console
2. Install maven plugin without restart  
  - `Manage Jenkins` > `Jenkins Plugins` > `available` > `Maven Invoker`
  - `Manage Jenkins` > `Jenkins Plugins` > `available` > `Maven Integration`

3. Configure maven path
  - `Manage Jenkins` > `Global Tool Configuration` > `Maven`