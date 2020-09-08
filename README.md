# Kubernetes-Jenkins-integration

# CI/CD Jenkins Kubernetes Integration:

## Prerequisites:
•	Basic knowledge of Jenkins

•	Basic knowledge of Ubuntu

•	Basic knowledge of AWS: EC2, IAM.
Amazon EC@ key pairs setup:  [ec2-key-pairs link](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

## Step 1:
### AWS instance creation: Jenkins Server
*	Login with your credentials to AWS cloud platform.
*	AWS console > Services > EC2 > Create instance > Launch Instance 
*	Choose an Amazon Image (AMI) > select eligible Ubuntu Server > Instance type> relative tier.
*	Configure instance details > configure VPC / IAM roles if required or otherwise leave defaults.
*	Storage, accept default and click next > add appropriate tags.
    Example: Jenkins Instance Server (JS).
*	Security Group > create a new security group

-	Security group name: jenkins-sg

-	Description: jenkins-sg

| Type          | Protocol      | Port Range  | Source      | Source   | 
| :-----------: |:-------------:|:-----------:|:-----------:| :------: |
| SSH           | TCP           | Anywhere    | 0.0.0.0/0   | ::/0     |
| Custom TCP    | TCP 8080      | Anywhere    | 0.0.0.0/0   | ::/0     | 


Type	Protocol	Port Range	Source	Source	Description
SSH	TCP	22	Anywhere	 0.0.0.0/0, ::/0	 
Custom TCP	TCP	8080	Anywhere	0.0.0.0/0, ::/0	 
	Review the instance details > click on ‘Launch’ 
	select an existing key pair or create a new pair > Jenkins.pem > download this key to pc >Launch Instance.
	Instance should start to spin-up or run.

Connect to your Instance:

	Select the Instance > click on “Connect” tab.
	Select the option I would like to connect with a standalone SSH client.
Copy the example: CLI and paste to connect

### Jenkins Server instance from command line:
ssh -i  "jenkins.pem" ubuntu@ec2-x.eu-east-1.compute.amazon.aws.com
ssh -i  "jenkins.pem" ubuntu@public-ip-public-dns 

move forward with typing “yes” to confirm.
Installation of Java:
Since Jenkins is a Java application, first step is to install Java. Update the package index and install Java 8 OpenJDK package with below commands.
sudo apt update
sudo apt install openjdk-8-jdk -y
Installation of Jenkins:
	Follow the below commands to create/add debian repository: Import the GPG keys of Jenkins repository:
sudo wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add –

	retype the same command to see the message with ‘ok’:
sudo wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add –
	package repository to server source list:
Add Jenkins repository to system with:
sudo sh echo deb https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
deb https://pkg.jenkins.io/debian-stable binary/

	run system update and Install jenkins
sudo apt-get update
sudo apt-get install Jenkins -y

	After Installation, check the status of Jenkins, start the service
sudo systemctl start Jenkins
sudo systemctl status Jenkins

	Jenkins running in port 8080 by default, allow the traffic to port.
Sudo ufw allow 8080
Sudo ufw status
Sudo ufw enable
	If you see the status in inactive run the enable command.
Sudo ufw status

Docker installation:
	install packages to allow apt to use a repository over HTTPS:
sudo apt-get update && sudo apt-get install -y apt-transport-https

	Install. Docker:
sudo curl -fsSL get.docker.com | /bin/bash

	Add Jenkins user to docker group:
sudo usermod -aG docker Jenkins

	Restart Jenkins
sudo systemctl restart jenkins
Connect to Jenkins site with port 8080:
	Copy the public ip address from AWS EC2 – Jenkins Server.
	Open a browser and paste this ip address followed by port 8080
 		example: http://public-ip:8080
	Jenkins page it should come up with ‘unlock Jenkins’ requires password.
	Get back to Jenkins Server ‘terminal window’ > 
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
	copy the password from the file to browser ’jenkins unlock’ page to enter in.
	select Install suggest plugin, should see next window with ‘Getting Started’.
	Wait until plugins are installed.
	In Create Admin User (May create or skip with continue as admin).
	Jenkins is ready!

Jenkins console:


## Step 2:

Setup of Kubernetes Cluster:

	Depending upon the work environment, process will remain same for master and worker nodes/instance.

Master Instance: 4GB RAM , 2 Core Processor
Worker Instance: 2GB or 1 GB RAM, 1 Core Processor.
	AWS Console> Services> EC2> Create instance > Launch Instance 
	Choose an Amazon Image (AMI) > select eligible Ubuntu Server > Instance type> relative tier with ‘2 Core processor, 4 GB RAM’ compatible.
	Configure instance details > configureVPC / IAM roles if required or otherwise leave defaults.
	Storage, accept default and click next > add appropriate tags.
    Example: Kubernetes Master (KM).
	Security Group > create a new security group
o	Security group name: KM-sg
o	Description: KM-sg


| Type          | Protocol      | Port Range  | Source      | Source   | 
| :-----------: |:-------------:|:-----------:|:-----------:| :------: |
| SSH           | TCP           | Anywhere    | 0.0.0.0/0   | ::/0     |
| Custom TCP    | TCP 8080      | Anywhere    | 0.0.0.0/0   | ::/0     | 


	Review the instance details > click on ‘Launch’ 
	select an existing key pair or create a new pair > KM.pem > download this key to pc >Launch Instance.
	Instance should start to spin-up or run.

Connect to your Instance:

	Select the Instance > click on “Connect” tab.
	Select the option I would like to connect with a standalone SSH client.
Copy the example: CLI and paste to connect.

Follow the same steps for Kubernetes worker instance creation with K-W-I.pem (key pair) and K-W-I-sg (Security group).

### Connect to Kubernetes Master & Worker instances from command line:
ssh -i  "KMpem" ubuntu@ec2-x.eu-east-1.compute.amazon.aws.com
ssh -i  "KM.pem" ubuntu@public-ip-public-dns 


with reference to Kubernetes Documentation page: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

Below commands are common for Master and Worker instance.
sudo apt-get update -y
sudo apt-get install -y apt-transport-https
sudo su –
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add –
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update -y
swapoff -a
sed -i '/swap / s/^\(.*\)$/#\1/g' /etc/fstab

	enable ip tables on Master and worker instance:

modprobe br_netfilter
sysctl -p
sudo sysctl net.bridge.bridge-nf-call-iptables=1

	Install Docker :
apt install docker.io -y
usermod  -aG docker ubuntu
systemctl restart docker
systemctl enable docker.service
	install kubernetes.
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

	Restarting the kubelet:
systemctl daemon-reload
systemctl restart kubelet
systemctl enable kubelet.service

Creating a cluster with kubeadm:
	To initialize the control-plane node run the below command in Master:
kubeadm init 

	switch to root user to run the below commands derived from init,

mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
	Check the status of node. Nodes status will be inactive, until CNI is configured.
kubectl get nodes

	Using Weave net CNI
kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')

	check the status of pods with weave net CNI created, node status should be active
 kubectl get pods –all-namespace
kubectl get nodes

	Join the master to worker node with token: copy the token command line to worker.
kubeadm token create --print-join-command

	Worker Instance: Paste the kubeadm join command 
sudo kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>

	Check the node status from Master instance command line:
kubectl get nodes

## Step 3:

### Setup Jenkins to Kubernetes:


	In Jenkins Browser, in the Jenkins dashboard and Click on ‘New Item’.
	Enter the Item name, in this case we have named it Mongo-docker-k8s. Choose the ‘Pipeline option’ and click ok.
	Install the Plugin from Manage Plugin manager > Available
o	add Maven > ‘install automatically > Maven-3.6.1 > apply > save
o	kubernetes continuous deploy > install without restart > ok
	Jenkins > Credentials > add> select from dropdown: GIT
o	Branch: master
o	Credentials: add > Jenkins > 
o	Add credentials: 
o	kind: username & password
o	scope: global
o	username: GitHub username
	Jenkins > Credentials > add > Build credentials to variables
o	Bindings: Secret text
o	kind: global
o	secret: docker password
o	ID: docker_hub_credentials
o	description: docker_hub_credentials
	Jenkins > Credentials > add > kubernetes configuration > 
o	Scope: Global 
o	ID: KUBERNETES_CLUSTER_CONFIG
o	Description: kubernetes config
o	kubeconfig:  Enter directly > paste content form “cat .kube/config” from master server.
	Specify the details of the job, mongo-docker-k8s > configure > pipeline >

node{
  stage("Git Clone"){

    git credentialsID: 'GIT_CREDENTIALS', url: 'https://github...'
  }

  stage("Maven Clean Build"){

    def mavenHome = tool name: "Maven-3.6.1", type: "Maven"
    def mavenCMD = "${mavenHome}/bin/mvn"
    sh "${mavenCMD} clean package"

  }

  stage("Build Docker Image"){
     sh "docker build -t dockerk8s/mongo-db-k8s . "
  }

  stage ("Docket Push"){
   withCredetials([string(credentialsID: 'docker_hub_credentials'......)]){
     sh "docker login -u dockerk8s -p ${docker_hub_credentials}"
   }
    sh "docker push dockerk8s/mongo-db-k8s "
  }
/**
  stage ("deploy application in k8s"){
     kubernetesDeploy(
         configs: 'BootMongo.yml',
         kubeconfigID: 'KUBERNETES_CLUSTER_CONFIG',
         enableConfigSubstitution: true
     )
  }
**/
  stage("Deploy to k8s cluster"){
    sh "kubectl apply -f 'BootMongo.yml"
  }

}

•	Click on “Build Now” to see the defined job have been successful.
•	Once the build is completed, check the status, has been successful. Click on #1 in history to see the details of build.
•	Click in Console Output to see details of build.


## Another Approach to Jenkin server:

•	install kubectl and add kubeconfig in Jenkins Server instance.
sudo apt-get update && sudo apt-get install -y apt-transport-https

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add –

echo "deb https://apt.kubernetes.io/kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt-get install -y kubectl

•	Login with Jenkins user: below commands will only work for jenkins user 
sudo -i -u jenkins

cd ~

mkdir .kube

•	Create the vi.kube/config file and paste content form .kube/config from Master server
vi .kube/config

•	Check the pods & services status:
kubectl get pods
kubectl get svc

•	Delete the replication controller, pods, services, to run new build from Jenkins console.
kubectl delete rc xxx
kubectl delete svc xxx
kubectl delete pods xxx

•	Run the Jenkins “Build now” and see the Console Output.
•	Check the below commands 
kubectl get pods
kubectl get svc


##  Jenkins:
Apart from the steps shown above there are different ways to create build job, the options available.

Jenkins such a fantastic continuous deployment tool.
