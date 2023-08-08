# Interactive Cares Task

This project demonstrates the process of setting up a Kubernetes cluster with Jenkins integration for continuous integration and deployment of a Node.js application. The goal of this project is to automate the deployment process, making it more efficient and reproducible.

## Project Overview

Modern software development practices emphasize automation, continuous integration, and deployment to streamline the development lifecycle. This project showcases a practical implementation of these principles by integrating Kubernetes, Jenkins, GitHub Actions, and GitHub in a cohesive DevOps pipeline.

## Project Steps

### Step 1: Create a Sample Node.js App

Begin by creating a simple Node.js application that will serve as the target application for deployment. This application can be as basic as a "Hello World" example or a more intricate application depending on your needs.

### Step 2: Create an EC2 Instance in AWS

Instance type : t2.medium (cpu= 2 core, ram= 4gb for kubernetes cluster minimum requirements)
Provision an Amazon EC2 instance on AWS to host both the Kubernetes cluster and Jenkins. Ensure that you select appropriate instance specifications to meet your project requirements.

### Step 3: Set Up Kubernetes Cluster

Install and configure Kubernetes on the EC2 instance. Utilize tools such as `kubeadm` to establish a functional Kubernetes cluster. Make sure to allocate appropriate resources to each Kubernetes component.
![Screenshot (10)](https://github.com/Mahtabhj/Interactive-cares-task/assets/48786676/3f2c0837-e3d5-4b41-90b9-ad6a26cace8e)


### Step 4: Run Jenkins as a Docker Container

Run Jenkins within a Docker container on the EC2 instance. This approach isolates Jenkins, facilitating its management and minimizing conflicts with other software components.

```cmd
 docker run -p 8080:8080 -p 50000:50000 -d -v /var/run/docker.sock:/var/run/docker.sock -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

### Step 5: Configure Jenkins

Configure Jenkins by installing the necessary plugins that facilitate integration with GitHub, Docker, and Kubernetes. These plugins empower Jenkins to communicate with each stage of your deployment pipeline effectively.
I install 2 plugins : 
                          1. Docker Pipeline
                          2. Kubernetes Deployment 1.0 (this is not available in jenkins , i use it by downloading it Here is the link for download : [click here](https://mirrors.tuna.tsinghua.edu.cn/jenkins/plugins/kubernetes-cd/1.0.0/kubernetes-cd.hpi) 

After that I setup 2 credentials : 1. DockerHub Credentials
                                 2. Kubernetes Credentials

### Step 6: Write Jenkins Pipeline

Craft a Jenkins pipeline script that outlines the deployment process. Your pipeline should include stages for checking out code from the GitHub repository, building Docker images, pushing images to a Docker registry, and deploying the application to the Kubernetes cluster.
![Screenshot (11)](https://github.com/Mahtabhj/Interactive-cares-task/assets/48786676/1dc9a772-8157-4464-a1cc-617cdb98bd93)


### Step 7: Set Up GitHub Webhook

Configure a webhook in your GitHub repository settings to trigger the Jenkins pipeline automatically whenever new code changes are pushed to the repository. This integration ensures that your deployment process remains responsive to code updates.

### Step 8: Implement GitHub Actions for Testing

Create a GitHub Actions workflow that runs unit tests on each push to the repository. This workflow ensures that the code changes do not break existing functionality and maintains the code's quality.



