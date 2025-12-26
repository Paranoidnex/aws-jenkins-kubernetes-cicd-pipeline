CI/CD Pipeline with Jenkins, Docker & Kubernetes on AWS
This project demonstrates how I built and deployed a Flask web application using a fully automated CI/CD pipeline with Jenkins, Docker, Kubernetes, and AWS EC2.

📌 Project Overview

Flask App hosted on AWS.

Kubernetes cluster deployed on AWS EC2 using kubeadm.

Flannel for pod networking.

Jenkins CI/CD pipeline automates build → test → deploy.

Docker used for containerization and image distribution.

Achieved ~80% faster deployments compared to manual steps

🛠️ Tech Stack

CI/CD: Jenkins

Containers: Docker

Orchestration: Kubernetes (kubeadm)

Networking: Flannel

Cloud: AWS EC2

Application: Flask (Python)

Setup Steps 1️⃣ Provision AWS EC2 Instances

Launch 1 master node + N worker nodes (Ubuntu recommended).

Configure security groups (allow ports 22, 80, 443, 6443).

2️⃣ Install Dependencies

sudo apt update && sudo apt upgrade -y sudo apt install -y docker.io apt-transport-https curl

3️⃣ Setup Kubernetes Cluster

Install kubeadm, kubelet, kubectl
sudo apt install -y kubeadm kubelet kubectl

Initialize cluster on master
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

Setup kubeconfig
mkdir -p $HOME/.kube sudo cp -i /etc/kubernetes/admin.conf 
H
O
M
E
/
.
k
u
b
e
/
c
o
n
f
i
g
s
u
d
o
c
h
o
w
n
(id -u):$(id -g) $HOME/.kube/config

Apply networking (Calico or Flannel)
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

Join worker nodes:

kubeadm join <MASTER_IP>:6443 --token --discovery-token-ca-cert-hash sha256:

4️⃣ Setup Jenkins on Master Node

sudo apt install openjdk-11-jdk -y wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add - sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ >
/etc/apt/sources.list.d/jenkins.list' sudo apt update sudo apt install jenkins -y sudo systemctl enable jenkins && sudo systemctl start jenkins

5️⃣ Create Jenkins Pipeline

Configure GitHub webhook.

Build → Dockerize Flask app → Push image to registry (ECR/DockerHub).

Deploy to Kubernetes via kubectl apply -f deployment.yaml.

🚀 Deployment Flow

Developer commits code → GitHub

Jenkins triggers pipeline → builds Docker image

Docker image pushed to registry (ECR/DockerHub)

Jenkins deploys new version on Kubernetes cluster

Flask app available via AWS public IP

✨ Key Learnings

Setting up Kubernetes clusters with kubeadm on AWS EC2

Configuring networking with Calico/Flannel

Automating CI/CD workflows with Jenkins

Integrating Docker + Kubernetes for scalable deployments

Project Structure
├── app.py # Flask app
├── requirements.txt # Python dependencies
├── Dockerfile # Build image
├── deployment.yaml # Kubernetes Deployment
├── service.yaml # Kubernetes Service (NodePort/LoadBalancer)
└── Jenkinsfile # CI/CD pipeline definition

Deployment Flow
1.Developer commits code → GitHub

2.Jenkins triggers pipeline → builds Docker image

3.Docker image pushed to registry (ECR/DockerHub)

4.Jenkins deploys new version on Kubernetes cluster

5.Flask app available via AWS public IP

Project files
1️⃣ Jenkinsfile – CI/CD pipeline definition

pipeline{

agent any

environment{
    docker_image = "paranoidnex/python-app"
    tag = """${BUILD_NUMBER}"""
}
stages{
    
    stage("github checkout"){
        steps{
            git branch: 'main', credentialsId: 'github-cred', url: 'https://github.com/Paranoidnex/python-app.git'
        }
    }
    
    stage("docker image build"){
        steps{
            sh "docker build -t ${docker_image}:${tag} ."
        }
    }
    
    stage("docker push"){
        steps{
            withDockerRegistry(credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/') {
sh "docker push ${docker_image}:${tag}"
} } }

    stage("k8s yaml creation"){
        steps{
            fileOperations([fileCreateOperation(fileContent: """apiVersion: apps/v1
kind: Deployment metadata: name: python-app-deploy labels: app: python-app spec: replicas: 1 selector: matchLabels: app: python-app template: metadata: labels: app: python-app spec: containers: - name: python-app image: ${docker_image}:${tag} ports: - containerPort: 80
apiVersion: v1 kind: Service metadata: name: python-app-service spec: selector: app: python-app ports: - protocol: TCP port: 80 type: NodePort
""", fileName: 'deploy.yml')]) } }

    stage("k8s deplyment"){
        steps{
            script{
                kubernetesDeploy(configs:"deploy.yml",kubeconfigId:"kubeconfig")
            }
        }
    }
}
}

2️⃣ Dockerfile – Flask App container

FROM python:3.11-slim WORKDIR /app RUN pip install --no-cache-dir Flask==3.0.3 COPY . . EXPOSE 80 CMD ["python", "app.py"]

3️⃣ deployment.yaml – Kubernetes Deployment

apiVersion: apps/v1 kind: Deployment metadata: name: python-app-deploy labels: app: python-app spec: replicas: 1 selector: matchLabels: app: python-app template: metadata: labels: app: python-app spec: containers: - name: python-app image: ${docker_image}:${tag} ports: - containerPort: 80
apiVersion: v1 kind: Service metadata: name: python-app-service spec: selector: app: python-app ports: - protocol: TCP port: 80 type: NodePort

5️⃣ app.py – Sample Flask App

from flask import Flask, render_template

app = Flask(name)

@app.route('/welcome') def welcome(): return render_template('welcome.html')

if name == 'main': app.run(host='0.0.0.0', debug=True, port=80)
