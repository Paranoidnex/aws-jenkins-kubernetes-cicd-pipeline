CI/CD Pipeline with Jenkins, Docker & Kubernetes on AWS
This project demonstrates how I built and deployed a Flask web application using a fully automated CI/CD pipeline with Jenkins, Docker, Kubernetes, and AWS EC2.

**📌 Project Overview**
Flask application hosted on AWS EC2
Kubernetes cluster deployed using kubeadm
Flannel used for Kubernetes pod networking
Jenkins CI/CD pipeline automates:
Build
Containerization
Deployment
Docker used for image creation and distribution
Achieved ~80% faster deployments compared to manual deployment

**🛠️ Tech Stack**
CI/CD -	Jenkins
Containers - Docker
Orchestration - Kubernetes (kubeadm)
Networking - Flannel
Cloud - AWS EC2
Application - Flask (Python)

**🏗️ Architecture Overview**
Developer → GitHub → Jenkins → Docker → Registry → Kubernetes → Flask App

**🚀 Deployment Flow**
Developer pushes code to GitHub
Jenkins pipeline is triggered automatically
Docker image is built and tagged
Image is pushed to DockerHub
Jenkins deploys the app to Kubernetes
Flask app becomes available via AWS public IP

**🧱 Setup Steps**
**1️⃣ Provision AWS EC2 Instances**
Launch:
1 Kubernetes master node
N worker nodes 
Configure Security Groups:
22 → SSH
80  → Application access

**2️⃣ Install Dependencies**
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io apt-transport-https curl

**3️⃣ Setup Kubernetes Cluster**
sudo apt install -y kubeadm kubelet kubectl
Initialize cluster on master: sudo kubeadm init --pod-network-cidr=192.168.0.0/16
Configure kubeconfig: mkdir -p $HOME/.kube,
                      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config,
                      sudo chown $(id -u):$(id -g) $HOME/.kube/config
Apply Flannel networking
Join worker nodes

**4️⃣ Setup Jenkins on Master Node**
sudo apt install openjdk-11-jdk -y

wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -

sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > \
/etc/apt/sources.list.d/jenkins.list'

sudo apt update
sudo apt install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins


**5️⃣ Create Jenkins Pipeline**
Pipeline stages:
GitHub checkout
Docker image build
Docker image push
Kubernetes deployment
GitHub webhook triggers Jenkins automatically.

**✨ Key Learnings**
Kubernetes cluster setup using kubeadm
Pod networking using Flannel
Jenkins CI/CD automation
Docker + Kubernetes integration
Real-world cloud deployment on AWS EC2
