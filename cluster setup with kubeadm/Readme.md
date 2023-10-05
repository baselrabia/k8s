<h1> Setup K8s Cluster using Kubeadm </h1> 

## Step 1: Turn off the swap & firewall </br>

	sudo swapoff -a
  

## Step 2: Set up Docker's Apt repository. </br>
### - Add Docker's official GPG key:
	 
	sudo apt-get update
	sudo apt-get install ca-certificates curl gnupg
	sudo install -m 0755 -d /etc/apt/keyrings
	curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
	sudo chmod a+r /etc/apt/keyrings/docker.gpg

### - Add the repository to Apt sources:
	
	echo \
	  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
	  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
	  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	sudo apt-get update

### - Install the Docker packages.

  	sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

### - Verify that the installation is successful by running the hello-world image:

	sudo docker run hello-world

## Step 3: Install cri-dockerd,
Note: Docker Engine does not implement the CRI which is a requirement for a container runtime to work with Kubernetes. For that reason, an additional service cri-dockerd has to be installed.
 cri-dockerd ->  https://github.com/Mirantis/cri-dockerd </br>

### - Install git 
	sudo apt install git

### - Install wget 
	sudo apt install wget

### - Clone cri-dockerd

	git clone https://github.com/Mirantis/cri-dockerd.git

### - Install golang 

	wget https://go.dev/dl/go1.21.1.linux-amd64.tar.gz
	sudo  tar -C /usr/local -xzf go1.21.1.linux-amd64.tar.gz
	export PATH=$PATH:/usr/local/go/bin
	go version

### - Install cri-dockerd
	cd cri-dockerd
	sudo apt install make
	make cri-dockerd
 
	sudo mkdir -p /usr/local/bin
	sudo install -o root -g root -m 0755 cri-dockerd /usr/local/bin/cri-dockerd
	sudo install packaging/systemd/* /etc/systemd/system
	sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
	sudo systemctl daemon-reload
	sudo systemctl enable --now cri-docker.socket

## Step 4: Installing kubeadm, kubelet and kubectl, install Kubernetes package repositories


### - Install k8s package

 	sudo apt-get update
	sudo apt-get install -y apt-transport-https ca-certificates curl
	curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
	echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

### - Install kubeadm 

	sudo apt-get update
 	sudo apt-get install -y kubelet kubeadm kubectl
 	sudo apt-mark hold kubelet kubeadm kubectl
 	
 	
 	
### - Get you node Ip address
  	ip addr
  	
### - Start your cluster using kubeadm - modify the ip addr with your ip
 
 	sudo kubeadm init --pod-network-cidr=10.128.0.10/16 --cri-socket=unix:///var/run/cri-dockerd.sock
   
### - To start using your cluster, you need to run the following as a regular user:

  	mkdir -p $HOME/.kube
  	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 	sudo chown $(id -u):$(id -g) $HOME/.kube/config


### - Check the nodes 

	 kubectl get nodes
	 
### - Since the master node is not ready, you need to setup your pod network addons (cni) </br> will use https://www.weave.works/

	kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

### - To watch the creating of the network pods 

	kubectl get pods -n kube-system --watch














 
