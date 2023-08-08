# Interactive Cares Task

This project demonstrates the process of setting up a Kubernetes cluster with Jenkins integration for continuous integration and deployment of a Node.js application. The goal of this project is to automate the deployment process, making it more efficient and reproducible.

## Project Overview

Modern software development practices emphasize automation, continuous integration, and deployment to streamline the development lifecycle. This project showcases a practical implementation of these principles by integrating Kubernetes, Jenkins, GitHub Actions, and GitHub in a cohesive DevOps pipeline.

## Project Steps

### Step 1: Create a Sample Node.js App

Begin by creating a simple Node.js application that will serve as the target application for deployment. This application can be as basic as a "Hello World" example or a more intricate application depending on your needs.

### Step 2: Writing a Dockerfile

To containerize my Node.js app, I need to create a Docker image using the following steps:

1. Create a file named `Dockerfile` (without any file extension) in the root directory of my project.

2. Add the following content to the `Dockerfile`:

```Dockerfile
FROM node:latest

WORKDIR /usr/src/app

COPY package.json ./

RUN npm install

COPY . .

EXPOSE 3000
CMD [ "node", "app.js" ]
```

## Step 3: Writing kubernetes manifest

To deploy the Node.js app using Kubernetes, i use this manifest file :


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodeapp-deployment
  labels:
    app: nodeapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodeapp
  template:
    metadata:
      labels:
        app: nodeapp 
    spec:
      containers:
      - name: nodeserver
        image: mahtabhj/newimage1:latest
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: nodeapp-service
spec:
  selector:
    app: nodeapp 
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 3000
    nodePort: 31110
```
This is a simple manifest file which contain 1 deployment and 1 service manifest

### Step 4: Create an EC2 Instance in AWS

Instance type : t2.medium (cpu= 2 core, ram= 4gb for kubernetes cluster minimum requirements)
Provision an Amazon EC2 instance on AWS to host both the Kubernetes cluster and Jenkins. Ensure that you select appropriate instance specifications to meet your project requirements.
<img width="482" alt="image" src="https://github.com/Mahtabhj/Interactive-cares-task/assets/48786676/63138d42-ecf6-4ce5-809c-4ac3c77a3da8">

### Step 5: Set Up Kubernetes Cluster

Install and configure Kubernetes on the EC2 instance. Utilize tools such as `kubeadm` to establish a functional Kubernetes cluster. Make sure to allocate appropriate resources to each Kubernetes component.
Procedure to set up kubernetes into EC2 : 
```cmd
sudo apt-get update
sudo apt-get upgrade -y
```
```cmd
sudo hostnamectl set-hostname "k8smaster.example.net"
```
```cmd
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```
```cmd
sudo modprobe overlay
sudo modprobe br_netfilter
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```
```cmd
sudo sysctl --system
sudo apt install -y curl software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
```
```cmd
echo "" | sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

```cmd
sudo apt update
sudo apt install -y containerd.io
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
```cmd
echo "" | sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
```cmd
sudo kubeadm init
```
```cmd
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl cluster-info
kubectl get nodes
```

If any problem occurs then run 
```cmd
sudo kubeadm reset
sudo kubeadm init
```

![Screenshot (10)](https://github.com/Mahtabhj/Interactive-cares-task/assets/48786676/3f2c0837-e3d5-4b41-90b9-ad6a26cace8e)


### Step 6: Run Jenkins as a Docker Container

Run Jenkins within a Docker container on the EC2 instance. This approach isolates Jenkins, facilitating its management and minimizing conflicts with other software components.

```cmd
 docker run -p 8080:8080 -p 50000:50000 -d -v /var/run/docker.sock:/var/run/docker.sock -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

### Step 7: Configure Jenkins

Configure Jenkins by installing the necessary plugins that facilitate integration with GitHub, Docker, and Kubernetes. These plugins empower Jenkins to communicate with each stage of your deployment pipeline effectively.

I install 2 plugins : 
1. Docker Pipeline
2. Kubernetes Deployment 1.0 (this is not available in jenkins , i use it by downloading it Here is the link for download : [click here](https://mirrors.tuna.tsinghua.edu.cn/jenkins/plugins/kubernetes-cd/1.0.0/kubernetes-cd.hpi) 
 <img width="900" alt="image" src="https://github.com/Mahtabhj/Interactive-cares-task/assets/48786676/4ec595a7-4233-433e-af27-af449c68a2bd">
 <img width="900" alt="image" src="https://github.com/Mahtabhj/Interactive-cares-task/assets/48786676/948bc4a3-dc98-42e5-8d74-7f9335e2d0dc">
  

After that I setup 2 credentials :
1. DockerHub Credential
2. Kubernetes Credential

![image](https://github.com/Mahtabhj/Interactive-cares-task/assets/48786676/ffe1962b-2d3e-4ed1-a0d9-b70cf29f3ebf)

### Step 8: Write Jenkins Pipeline

Craft a Jenkins pipeline script that outlines the deployment process. Your pipeline should include stages for checking out code from the GitHub repository, building Docker images, pushing images to a Docker registry, and deploying the application to the Kubernetes cluster. The pipeline I use is in the jenkins folder of this repository .
![Screenshot (11)](https://github.com/Mahtabhj/Interactive-cares-task/assets/48786676/1dc9a772-8157-4464-a1cc-617cdb98bd93)


### Step 9: Set Up GitHub Webhook

Configure a webhook in your GitHub repository settings to trigger the Jenkins pipeline automatically whenever new code changes are pushed to the repository. This integration ensures that your deployment process remains responsive to code updates.
<img width="311" alt="image" src="https://github.com/Mahtabhj/Interactive-cares-task/assets/48786676/e7011320-1a07-47c8-abed-ea07248e2f77">


### Step 10: Implement GitHub Actions for Testing

Create a GitHub Actions workflow that runs unit tests on each push to the repository. This workflow ensures that the code changes do not break existing functionality and maintains the code's quality. I use a sample unit test. When a push occurs in this repo the workflow will trigger and run that unit test .
<img width="323" alt="image" src="https://github.com/Mahtabhj/Interactive-cares-task/assets/48786676/7472e5a6-0878-4033-b297-9c3eae7588d6">



