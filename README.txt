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
	By default the ec2 machine has Java 1.7.* installed, remove it by 
	#yum remove java-1.7.*
	
	
	Check angain java version, you get nothing #java -version
	Now install java-1.8* as 
	#yum install java-1.8*
	
	Now you can check java version as 
	#java -version and will get java-1.8*
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
		cd ~/opt/maven
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
	change permission to execute to startup.sh and shutdown.sh
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
	 Check the public ip :8080/webapps
	 
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
	 Now check if tomcat is running on public-ip:8080
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
	kubernernetes-master node ip
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
		
--------------------------------------------------------
Deploy on Kubernetes
		Setup Kubernetes environment
		Use Kubectl command
		Write deployment and service files
		Create Ansible playbooks to deploy
		Run a Jenkins job
--------------------------------------------------------
Setup Kubernetes Clusters on AWS
	Goto AWS Console and Select EC2 -> Launch Instance
		Select Ubuntu Server 18.04 LTS
		select t2.micro ->Next ->Next ->Next
		Add Tag
		Name K8s-Management-Server
		Next
		Use Existing Security Group
		Review and Launch
		Launch
		Use Exiting Key Pair
		Accept the terms
		Launch Instances
		
	Now login into the machine using ssh
	$$ssh -i key ubuntu@ip
	ubuntu@ip$sudo su -  (become root user)	
	
	Now download the AWS CLI Bundle
	#curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
	Install unzip and python, by default it is not available
	#apt install unzip python
	Unzip AWS CLI Bundle
	#unzip awscli-bundle.zip
	
	Now copy the file under /usr/local/bin/aws
	#./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
	Now you can get the aws version
	#aws --version
	
	Now install kubectl
	#curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
	#chmod +x ./kubectl
	 #sudo mv ./kubectl /usr/local/bin/kubectl
	Now you check kubectl
	#kubectl
	
	
	Now install kops
	#curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
 	#chmod +x kops-linux-amd64
 	#sudo mv kops-linux-amd64 /usr/local/bin/kops
	Now you can check kops
	#kops
	
	
	Now Create an IAM user/role with Route53, EC2, IAM and S3 full access for EC2 instance
		Select IAM service -> Create Role> Select EC2 > Role Name : k8s-role, add s3full, ec2full, route53full, iamfull 
		Select K8 Management Server -> Instance Settings > Attach/Replace IAM Role > select created IAM role from dropdown list and apply
		
	Now configure aws
		#aws configure
		Select Default Zone us-east-1
		(You can get the zone name next to your login at the right top)
	
	Now create Private Route53 hosted zone 
		Select Route 53 service
		Select DNS Management ->get started now -> create hosted zone
		devopstest.net
		us-east-1
		And Availability Zone you can find, at the instance page (us-east-1c) 
		
	
	Now create s3 bucket
		From commandline run the following command
		#aws s3 mb s3://demo.k8s.devopstest.net
		You can go and check it is created into S3
	
	Now expose environment variable for Kops
	#export KOPS_STATE_STORE=s3://demo.k8s.devopstest.net
		
	Now create key-pair to login to the cluster (generate the key-pair cluster )
		#ssh-keygen
		#cd .ssh
	Now create kubernetes cluster definition on s3 bucket
	#kops create cluster --cloud=aws --zones=us-east-1a --name=demo.k8s.devopstest.net --dns-zone=devopstest.net --dns private 
		
	Output:
	Cluster configuration has been created.

	Suggestions:
	 * list clusters with: kops get cluster
	 * edit this cluster with: kops edit cluster demo.k8s.devopstest.net
	 * edit your node instance group: kops edit ig --name=demo.k8s.devopstest.net nodes
	 * edit your master instance group: kops edit ig --name=demo.k8s.devopstest.net master-ap-northeast-2b

	Finally configure your cluster with: kops update cluster --name demo.k8s.devopstest.net --yes
	
	Now create kubernetes cluster
	#kops update cluster demo.k8s.devopstest.net --yes
	
	Output of the above command
	Cluster is starting.  It should be ready in a few minutes.

	Suggestions:
	 * validate cluster: kops validate cluster
	 * list nodes: kubectl get nodes --show-labels
	 * ssh to the master: ssh -i ~/.ssh/id_rsa admin@api.demo.k8s.devopstest.net
	 * the admin user is specific to Debian. If not using Debian please use the appropriate user based on your OS.
	 * read about installing addons at: https://github.com/kubernetes/kops/blob/master/docs/operations/addons.md.
	
	
 Now validate the cluster (repeat the command till you see it ready)
 	   # kops validate cluster
	   
Now login to master node
		#ssh -i ~/.ssh/id_rsa admin@api.demo.k8s.devopstest.net  from the K8s-Master-Server and the sudo su -
		check the private ip from the instance of the master-node
		#exit
		#kubectl get nodes
In case you want to delete the kubernetes cluster
	Delete cluster
		# kops delete cluster demo.k8s.devopstest.net --yes
			
Now login to the kubernetes master node server
	#ssh -i ~/.ssh/id_rsa admin@api.demo.k8s.devopstest.net
	#sudo su -  [login as root user]
	#kubectl get nodes [one master and two nodes]
	#kubectl get pods [get pods if created]
	#kubectl get deploy [get deployment if created]
	
Deploying Nginx pods on Kubernetes
	Deploying Nginx Container

	#kubectl run sample-nginx --image=nginx --replicas=2 --port=80 [run two pods of nginx by pulling image from docker and name it as simple-nginx]
# kubectl run simple-devops-project --image=ravigkl/simple-devops-image --replicas=2 --port=8080 [ in case of ravigkl/simple-devops-image from docker hub]
	Execute the commands 
	#kubectl get pods
	#kubectl get deployments
	
Expose the deployment as service to access the application publically. This will create an ELB in front of those 2 containers and allow us to publicly access them.

	#kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
# kubectl expose deployment simple-devops-project --port=8080 --type=LoadBalancer
	#kubectl get services -o wide	
		
Now create the deploy and service yml files to deploy the image and expose it to access.
	#kubectl apply -f devops-test-deploy.yml
	#kubectl apply -f devops-test-service.yml

Integrate Kubernetes with Ansible
	Login to ansible server and copy public key onto kubernetes cluseter master account
	$ssh -i key ec2-user@public-ip
	then sudo su - ansibleadmin
	then cd .ssh
	cat id_rsa.pub
	copy it to the end of the K8s-Management-Server into ~/.ssh/authorized_keys
	It is added under root account of the K8s-Management-Server, so you can use it to login from the ansible server to to master-node using the public ip of master node
	Note: the user is admin
	from Ansible server as ansibleadmin user
	$ssh -i ~/.ssh/id_rsa  admin@public_ip_of_master_node - it will login to master node as admin and you can change it to root as 
	$sudo su - 
	#cat >>authorized_keys [append by copying the public key of the ansible server to this file]
	
	Now into ansible server, create a folder kubernetes under /opt
	$sudo mkdir kubernetes
	$cd kubernetes
	$ls [nothing is there]
	Create hosts file
	$sudo vi hosts
	[ansible-server]
	localhost
	
	[kubernetes]
	master node public ip
	
	save it :wq
	Now create the deployment and service files into kubernetes folder where hosts file is
	$sudo vi deployment and service files
	
	By deleting the deployment, it deletes deployment and pods but the service attached to it
	
	Delete all the service and deployment onto kubernetes master node
	Now fire the ansible-playbook script on the ansible server
	
	
	Now let's execute the Kubernetes deployment and service file into ansible-playbook from Jenkins Server
	Before this process, just delete the previously deployed deployment and service
	Goto the Jenkins Server, Create New Item Deploy_On_Kubernetes_using_Ansible free style project without source but only post build action
	Send build artifacts over SSH Select Ansible_Server, and enter the executable playbook command along with hosts and file location
	ansible-playbook -i /opt/kubernetes/hosts /opt/kubernetes/kubernetes-devops-test-deploy.yml;
	ansible-playbook -i /opt/kubernetes/hosts /opt/kubernetes/kubernetes-devops-test-service.yml;
	
	Now you can build the image of the war from the Github source code for the Kubernetes CI
	Move the Dockerfile, create-devops-image from /opt/docker to /opt/kubernetes
	
	Change the ownership of these files to ansibleadmin instead of root
	$ls -ld /opt/kubernetes
	$sudo chown -R ansibleadmin:ansibleadmin /opt/kubernetes
	Also change the hosts from all to ansible-server
	
	Now CI/CD Pipeline using GitHub, Jenkins, Ansible, DockerHub, Kubernetes
	Edit the CI Job and Choose Build Other Project from the Post Build Action and select the CD job and trigger it when CI build is stable
	
	Kubernetes supports rolling deployment of the images using the version control of the images
	And creates new container with the new image
	

	Update hosts file with new group called kubernetes and add kubernetes master in that.

	Create ansible playbooks to create deployment and services

	Check for pods, deployments and services on kubernetes master

	kubectl get pods -o wide 
	kubectl get deploy -o wide
	kubectl get service -o wide
	Access application using service IP

	wget <kubernetes-Master-IP>:31200
