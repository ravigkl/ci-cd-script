Jenkins Server deployment - 
	Choose EC2 instance for t2-micro Linux AMI with Java, intance name: Jenkins_Server
	Create Security Group - Devops_Project_SG
	Add new rule for port 8080 under TCP 0.0.0.0/0,::/0 (this network setting is not recommended to open for all, but for specific range of ip)
	Create KeyPair - Deveops_Project, download the private key
	chmod 600 *.pem (the the key executable)
	Copy the public IP of the instance - 18.208.149.120

	Use ssh -i <privatekey> ec2-user@p

	After login become the root user as $sudo su -
	For installation, you should have root privileges
	Check for java version, #java -version
	By default the ec2 machine has Java 1.7.* installed, remove it by #yum remove java-1.7.*
	Check angain java version, you get nothing #java -version
	Now install java-1.8* as yum install java-1.8*
	Now you can check java version as #java -version and will get java-1.8*
	Now set the JAVA_HOME by searching the location of jre 
	#find /usr/lib/jvm/java-1.8* | head -n 3
	JRE is located at /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.50.amzn1.x86_64
	Set the JAVA_HOME into the home directory
	#cd ~
	#vi .bash_profile
	JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.50.amzn1.x86_64
	PATH=$PATH/bin:$JAVA_HOME
	:wq (save it)
	#echo $JAVA_HOME - which will not give the correct path
	logout and the login and check #echo $JAVA_HOME
	Now search for jenkins download into google
	We have two options- LTS and Weekly
	It is recommended as best practice to install LTS
	Choose redhat fedora link and download the below to install jenkins
	sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
	sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
	yum install jenkins
	Now you can check the #service jenkins status
	you will get jenkins is stopped
	Now you can use #service jenkins start to make jenkins running
	Now you can open into browser to login into jenkins
	http://18.208.149.120:8080
	you will get the inital password here /var/lib/jenkins/secrets/initialAdminPassword
	Skip the first page and start using jenkins
	
	Change admin password of the Jenkins - go to admin and click on Configure, change the password, appy,save and relogin to verify
	Now configure the JAVA_HOME into Jenkins, go to Jenkins->Manage Jenkins->Global Tool Configuration
	Set the JAVA_HOME into JDK and copy the path from the outout of #echo $JAVA_HOME, Apply and Save
	
	
	Now we can write the first Jenkins job
	Click on New Item, provide Item name, choose freestyle project, click on OK button
	You will see tabs - General, Source Code Management, Build triggers	, Build, Post-Build Actions
	In build section, select shell and write there echo "Welcome to my first Project Build"
	Apply, Save and then click on Build now to run the build for execution of the script
	
	Click on Build #1 and then click on Console Output
	
	Git setup and install
	Install Git on jenkins_server by login into it #yum install git -y
	
	You can change the hostname instead of ip address
	$hostname
	$sudo su -
	#yum install git -y
	
	Setup git into jenkins console
	Manage Jenkins > Manage Plugins > available > github
		From Available , search for github and select it and click install without restart
	
	Configure git path
		Manage Jenkins > Global Tool Configuration > git
		name - github
		path /usr/bin/git
		Apply, Save
	
	Maven setup on jenkins server
		Download the maven package into /opt/maven
		mkdir /opt/maven
		wget fileurl
		tar -xvzf apache-maven-*.bin.tar.gz
	Setup M2_HOME and M2 paths into .bash_profile and use this to the path variable
	M2_HOME=/opt/maven/apache-maven-3.6.1
	M2=$M2_HOME/bin
	PATH=Existing_Path:$M2_HOME:$M2
	
	logoff and login and check #mvn -version
	
	Now setup the maven plugin from Jenkins Console
		Manage Jenkins > Manage Plugins > available > Maven Invoker
		Manage Jenkins > Manage Plugins > available > Maven Integration
	Configure maven path
		Maven Jenkins > Global Tool Configuration > Maven
	
	Now choose new item
		Name My_First_Java_Project
		Source Code Management
			Git: Repository URL: copy and paste the url from github
			Branch: master
			Credentials: None for public repository
			Repository browser: Auto
		Build
			Root POM pom.xml
			Goals and Options: clean install package
	Setup tomcat server - install new EC2 instance
	use the same security group
	ssh -i key ec2-user@3.89.200.190
	cd /opt
	download tomcat 8
	wget http://apachemirror.wuchna.com/tomcat/tomcat-8/v8.5.53/bin/apache-tomcat-8.5.53.tar.gz
	tar -xzvf apache-tomcat-8.5.53.tr.gz
	change permission to executrabel to startup.sh and shutdown.sh
	chmod +x /opt/apache-tomcat-8.5.53/bin/startup.sh
	chmod +x /opt/apache-tomcat-8.5.53/bin/shutdown.sh
	startup.sh
	Edit the context.xml file to access the manage app
	#find / -name context.xml
	edit the file /opt/apache-tomcat-8.5.53/webapps/manager/META-INF/context.xml
	comment the loopback value
	<!-- kkkkk. -->
	do the samething on other file
	/opt/apache-tomcat-8.5.53/webapps/host-manager/META-INF/context.xml
	shutdown and startup the server
	
	Now update the user information in the tomcat-users.xml file
	<role rolename="manager-gui"/>
	 <role rolename="manager-script"/>
	 <role rolename="manager-jmx"/>
	 <role rolename="manager-status"/>
	 <user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
	 <user username="deployer" password="deployer" roles="manager-script"/>
	 <user username="tomcat" password="s3cret" roles="manager-gui"/>
	 
	 restart the service
	 A user deployer with password deployer under manager-script role is allowed to do the deployment
	 access for the gui console with tomcat user and s3cret password under manager-gui role
	 
	 Deploy war file from Jenkins to EC2 Instance
	 for deployment from the Jenkins, install the plugin deploy to container install without restart
	 Once the build is successful, the Post-build Actions is Deploy ear/ear to container
	 Choose the Jenkins and setup the deployment user credentials
	 Apply and Save
	 Run the Build
	 check the url http://3.89.200.190:8080/webapp - Hola!!!
	 
	 Set the Build trigger whenever there is code commit by Polling SCM as cronjob setting
	 * * * * * (every minute, every hour, every day, every month, every week)
	 
	 Limitation: here we are using Jenkins for build as well as deploy
	 
	 Create another machine for the Docker instance on EC2 - Linux AMI 2018
	 Then goto the command line via ssh
	 #yum install docker
	 Then run docker
	 #docker ps
	 #service docker status
	 #service docker start
	 
	 Now goto Docker Hub and search for the Tomcat official image
	 https://hub.docker.com
	 register here yourself to upload your images, images are available as public and you can store it either private or public
	 #docker pull tomcat:latest
	 #docker images ls
	 Now create a container for the tomcat
	 #docker run --name tomcat-container -p 8080:8080 tomcat:latest
	 Now check if tomcat is running on http://3.89.31.76:8080
	 You will see 404 Not Found error
	 Now stop the container, ctrl+c
	 Remove the container #docker rm <container_id>
	 Check the container which is in stop state #docker ps -a
	 Run tomcat into container in detached mode as 
	 #docker run -d --name tomcat-container -p 8080:8080 tomcat:latest
	 Stop container before removing it
	 #docker stop <container_id>
	 
	 To resolve any issue inside the container, login into it as
	 #docker exec -it tomcat-container /bin/bash
	 
	 To fix the 404 error, you need to put webapp into webapps folder, since it is empty
	 For immediate fix, we can copy the webapps.dist folder contents into webapps folder
	 #cp -R * ../webapps
	 
	 Or you can use the 8.0 instead of latest version of tomcat
	 #docker pull tomcat:8.0
	 #docker run -d --name tomcat-8 -p 8081:8080 tomcat:8.0
	 Note: port 8081 should be opened into Security group configuration, if not add it to access the tomcat
	 This will fix the issue of 404 instead of doing copy of webapps.dist into webapps
	 
	 Integrate Docker with Jenkins
	 1. install a plugin Publish Over SSH (send build artifacts over SSH)
	 2. Store the credential of the dockerhub host into jenkins server
	 3. Create a user in docker as 
	 	#useradd dockeradmin
		#passwd dockeradmin
		This user should be part of docker group
		#usermod -aG docker dockeradmin
		Reconfirm to which group dockeradmin belongs
		#id dockeradmin
	4. Provide this credentials into Jenkins under Configure System -> SSH Servers (add)
	5. Enable password based authentication into docker host server, by default it is key based
		Edit the file #vi /etc/ssh/sshd_config
		Set PasswordAuthentication yes
		After changes restart the sshd service
		#service sshd reload
		Now test again, it should be successful.
	
	Deploy the war file onto the Docker Host container of tomcat
		1. Use SSH server
		2. Set the path for the source war file from the Jenkins Server
		3. Goto Docker server #su - dockeradmin #pwd #ls 
	
	Deploy the war file onto the Docker Tomcat Container
		1. Create a Dockerfile under root folder of the project
		#vi Dockerfile
		FROM tomcat:latest
		
		MAINTAINER Ravi S Singh
		
		COPY ./webapp.war /usr/local/tomcat/webapps
		
		save it and now run as 
		#docker build -t devops-project .
		#docker images
		Delete the previous tomcat container using 8080 port
		Build the new container using the docker-project image
		#docker run -d --name devops-container -p 8080:8080 devops-project
		Now check the application into the browser with ip:8080/webapp
		
	When you mention the execution script into Publish over SSH
	cd /home/dockeradmin;docker build -t devops-image .; docker run -d --name devops-container -p 8080 devops-image
	
	It will through error on click of build 
	docker: Error response from daemon: Conflict. The container name "/devops-container" is already in use by container 
	
	Here the Ansible comes for help to resolve this situation.
	Else manually you have to remove the webapp.war, remove image, remover container then the build would be successful
	Drawback of Jenkins working as Deployer
	1. Tedious work to register target machine where deployment is to be pushed
	2. Tedious job to clean the inventory before deployment onto the target machine
	Let's use Jenkis as build tool

	Setup Ansible onto EC2 Linux AMI 2018, by create instance, name Ansible-Server, assign existing SG and Keypair
	ssh -i key ec2-user@ip
	#yum install python
	#python --version
	#yum install python-pip
	#pip install ansible
	#ansible --version
	Note: the config-file: None, it is required to be set and host file to be created
	So you have to create a directory
	#mkdir /etc/ansible
	#useradd ansibleadmin
	#passwd ansibleadmin
	Add this user to sudoer group
	#visudo
	Goto end of file shif+g
	esc-o
	ansibleadmin ALL=(ALL)	NOPASSWD: ALL
	save
	
	You also need to install the docker here on Ansible Server
	#yum install docker -y
	#service docker start
	#service docker status
	
	Add the ansibleadmin use to docker group
	#usermod -aG docker ansibleadmin
	Enable the password authentication into Ansible Server instance
	#vi /etc/ssh/sshd_config
	and reload the sshd service
	#service sshd reload
	Now generate keys for ansibleadmin user
	#su - ansibleadmin
	#ssh-keygen
	By default the key generated is stored into .ssh folder ~/.ssh
	there are two keys
	a. private key: id_rsa
	b. public key: id_rsa.pub
	This public key needs to be copied to the file where you want to communicate
	
	Now create the same user- ansibleadmin into Docker Server and give the same password
	copy the key from ansible server to the docker server using below command from ansible server
	#ssh-copy-id ansibleadmin@docker-server-ip
	say yes and enter password of the ansibleadmin set into the docker host
	Now try logging into the machine docker host, with:   "ssh 'ansibleadmin@172.31.95.76'"
	and check to make sure that only the key(s) you wanted were added.
	Now connection and commuincation between docker and ansible established and verified
	
	Now create hosts file in ansible server /etc/ansible/hosts
	Into the file enter the ip of the machine where ansible will push the code for deployment, docker server
	#sudo vi /etc/ansible/hosts
	172.31.95.76
	localhost
	:wq
	#ansible all -m ping
	Copy the key to the localhost as well
	#ssh-copy-id localhost
	
	Now setup another SSH into Jenkins for the Ansible
	Create a directory into ansible server as /opt/docker
	#cd /opt
	#sudo mkdir docker
	#sudo chown -R ansibleadmin:ansibleadmin /opt/docker
	#cd docker
	#pwd
	/opt/docker
	Copy this path to the Repository location for the artifact storage
	Jenkins does not accept single / so use // while making the entry
	//opt//docker
	
	
	cd /home/dockeradmin; docker build -t devops-image .; docker run -d --name devops-container -p 8080:8080 devops-image
	Copy the Dockerfile from the docker host  dockeradmin to ansibleadmin
	$vi Dockerfile
	FROM tomcat:latest

	MAINTAINER Ravi S Singh

	COPY ./webapp.war /usr/local/tomcat/webapps
	:wq
	Now create an ansible file for creating image for deployment
	
	Create an ansible playbook, named create-devops-image.yml
	
	 # Option-1 : Createting docker image using command module 
	 ---
	 - hosts: all
	   become: true
	   tasks:
	   - name: building docker image
	     command: docker build -t simple-devops-image .
	     args:
	       chdir: /opt/docker

	 # option-2 : creating docker image using docker_image module 

	 #  tasks:
	 #  - name: building docker image
	 #    docker_image:
	 #      build:
	 #        path: /opt/docker
	 #      name: simple-devops-image
	 #     tag: v1
	 #     source: build
	
	 Also create a hosts file inside /opt/docker with only content localhost
	 This will be used just to store the build image to be deployed on all the remote hosts machines
	 Check if playbook is written correctly before execution
	 #ansible-playbook -i hosts simple-devops-image.yml --check		
	 If everything is fine, then run the ansible-playbook without check
	 
	 Now remove container and images from the ansible server
	 Goto jenkins server and update the Deploy_On_Ansible job
	 enter the Exec command
	 ansible-playbook -i /opt/docker/hosts /opt/docker/simple-devops-project.yml;
	 
	 Edit the simple-devops-project.yml and add the task to stop, and remove the container and image
	 
	 # Option-1 : Createting docker image using command module 
	 ---
	 - hosts: all
	   become: true
	   tasks:
	   - name: stop the running container
	     command: docker stop simple-devops-container
		 ignore_errors yes
		 
	   - name: remove the stopped container	 
	     command: docker rm simple-devops-container
		 ignore_errors yes
		 
	   - name: remove the image
	     command: docker rmi simple-devops-image
		 ignore_errors yes
		 	 
	   - name: building docker image
	     command: docker build -t simple-devops-image .
	     args:
	       chdir: /opt/docker
	 
Automate the deployment of the images push and pull from the dockerhub using Ansible Playbook
To push into the dockerhub, you should first login into docker hub, before executing the ansible playbook from ansibleadmin user as su - ansibleadmin.
#docker login	 
username
password
the become value should be false
Now add the docker server IP into the hosts file of the ansible server at /opt/docker
		
	
	
	
	
	
