# Install Jenkins on AWS EC2

Jenkins is a self-contained Java-based program, ready to run out-of-the-box, with packages for Windows, Mac OS X and other Unix-like operating systems. As an extensible automation server, Jenkins can be used as a simple CI server or turned into the continuous delivery hub for any project.

### Prerequisites
1. EC2 Instance 
   - With Internet Access
   - Security Group with Port `8080` open for internet

1. Java v11.x.x.x 

## Install Java
1. We will be using open java for our demo, Get the latest version from http://openjdk.java.net/install/
   ```sh
   amazon-linux-extras list | grep java
   sudo amazon-linux-extras -y install java-openjdk11
   ```

1. Confirm Java Version and set the java home momentally
   ```sh
   java -version
   find /usr/lib/jvm -name "*jre*"
   JAVA_HOME=/usr/lib/jvm/jre-11-openjdk-11.0.18.0.10-1.amzn2.0.1.x86_64
   PATH=$PATH:$JAVA_HOME
   ```
   
 ### To set it permanently update your .bash_profile
   ```sh
   vi ~/.bash_profile
   # If for some reason you don't find the PATH variable in this file on-premise, please add these lines along with the JAVA_HOME variable you're trying to setup.
   JAVA_HOME=/usr/lib/jvm/jre-11-openjdk-11.0.18.0.10-1.amzn2.0.1.x86_64
   PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME
   export PATH

   - To load your recent modifications, run
   
   [root@~]# source .bash_profile

   Then
   [root@~]# java -version
   
   The output should be something similar like this one below:

   openjdk version "11.0.18" 2023-01-17 LTS
   OpenJDK Runtime Environment (Red_Hat-11.0.18.0.10-1.amzn2.0.1) (build 11.0.18+10-LTS)
   OpenJDK 64-Bit Server VM (Red_Hat-11.0.18.0.10-1.amzn2.0.1) (build 11.0.18+10-LTS, mixed mode, sharing)
   ```

## Install Jenkins
 You can install jenkins using the rpm or by setting up the repo. We will set up the repo so that we can update it easily in the future.
1. Get the latest version of jenkins from https://pkg.jenkins.io/redhat-stable/ and install
   ```sh
   yum -y install wget
   sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
   sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key 
   sudo yum -y install jenkins

   # Note: The "epel-release" package install by the amazon package manager will help in getting the "daemonize" package required as a dependency by the "jenkins" package.
   ```


   ### Start Jenkins
   ```sh
   # Start jenkins service
   sudo service jenkins start OR sudo systemctl start jenkins

   # Setup Jenkins to start at boot,
   sudo chkconfig jenkins on OR sudo systemctl enable jenkins
   ```

   ### Accessing Jenkins
   By default jenkins runs at port `8080`, You can access jenkins at
   ```sh
   http://YOUR-SERVER-PUBLIC-IP:8080
   ```
  #### Configure Jenkins
- The default Username is `admin`
- Grab the default password 
- Password Location:`/var/lib/jenkins/secrets/initialAdminPassword`
- `Skip` Plugin Installation; _We can do it later_
- Change admin password
   - `Admin` > `Configure` > `Password`
- Configure `java` path
  - `Manage Jenkins` > `Global Tool Configuration` > `JDK`  
- Create another admin user id

### Test Jenkins Jobs
1. Create “new item”
1. Enter an item name – `My-First-Project`
   - Chose `Freestyle` project
1. Under the Build section
	Execute shell: echo "Welcome to Jenkins Demo"
1. Save your job 
1. Build job
1. Check "console output"

