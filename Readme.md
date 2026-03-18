
---

# 🚀 TREND APPLICATION DEPLOYMENT (DEVOPS PROJECT)

---

## 📁 Project Structure

```
Trend

├─ README.md

└─ dist

   ├─ index.html

   ├─ assets

   └─ vite.svg
```

---

## 📌 Understanding the Application

What you saw:

```
dist/

index.html

assets/
```

These are static website files.

So instead of NodeJS we will use a web server like Nginx to serve them.

---

# 🚀 DAY 1 — DOCKER + DOCKERHUB

---

## 🚀 Step 1 — Clone the Repository

Open terminal in VS Code or PowerShell:

```
git clone https://github.com/Vennilavanguvi/Trend.git

cd Trend
```

---

## 📁 Create Dockerfile

### Where

Inside the Trend project root folder:

```
Trend

├─ dist

└─ Dockerfile
```

### Tool

Docker

### Why we use Docker

To package the application with its runtime so it can run anywhere.

---

## 📝 Create Dockerfile

Open terminal:

```
notepad Dockerfile
```

```
FROM nginx:latest

COPY nginx.conf /etc/nginx/conf.d/default.conf

COPY dist/ /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---

## 📖 Understand the Dockerfile

### Base Image

```
FROM nginx:latest
```

Use official Nginx image from DockerHub.

### Copy Nginx Config

```
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

Replace default nginx config with our config.

### Copy Application

```
COPY dist/ /usr/share/nginx/html
```

Move website files into nginx web directory.

### Expose Port

```
EXPOSE 3000
```

Tell Docker that the container will run on port 3000.

### Start Nginx

```
CMD ["nginx", "-g", "daemon off;"]
```

Start nginx when container runs.

---

## 🚀 Step 3 — Build Docker Image

```
docker build -t trend-app .
```

output:

Successfully built
Successfully tagged trend-app

Check image:

```
docker images
```

You should see:

trend-app

---

## 🚀 Step 4 — Run Docker Container

```
docker run -d -p 3000:80 react-deploy-app
```

Explanation

Option Meaning

-d run container in background
-p 3000:80 map port 3000 to container port 80

---

## 🌐 Step 5 — Open Application

```
http://localhost:3000
```

---

## 📊 Step 6 — Verify Container

```
docker ps

docker images
```

---

## 🚀 Step 7 — Push Image to DockerHub

```
docker login
```

Tag Image

```
docker tag react-deploy-app lakshmanhari/trend-app
```

Push

```
docker push lakshmanhari/trend-app
```

Test Pull

```
docker pull lakshmanhari/trend-app
```

---

## 📌 DevOps Flow

```
React App (dist)

↓

Docker Image

↓

Docker Hub
```

---

## 📁 Correct Folder Structure

```
Trend/

├── dist/

├── nginx.conf

├── Dockerfile

└── README.md
```

---

## 📝 Create nginx.conf

```
notepad nginx.conf
```

```
server {

listen 80;

location / {

root /usr/share/nginx/html;

index index.html index.htm;

try_files $uri $uri/ /index.html;

}

}
```

---

## Explanation

Why we do this

This file tells Nginx:

Run server on port 3000

Serve files from /usr/share/nginx/html

If route not found → return index.html

dist/
This contains the production React build (HTML, CSS, JS).

Nginx will serve these files.

nginx.conf
Configuration file for the Nginx web server.

Tells Nginx to run on port 3000 and serve the dist files.

Dockerfile
Instructions for Docker to build the container image.

---

# 🚀 DAY 2 — TERRAFORM + AWS + EKS

---

## Step 1 — Login to DockerHub

```
docker login
```
It will ask:

Username:

Password:

Enter your DockerHub username and password.

If login is successful you will see something like:

Login Succeeded

---

## Check Your Local Image

```
docker images
```

You should see:

trend-app

output

REPOSITORY TAG IMAGE ID

trend-app latest 7d8f9ab23
---

## Verify on DockerHub

Go to:

[https://hub.docker.com](https://hub.docker.com)

Open Repositories.

You should see:

trend-app

---

## Terraform setup

```
terraform -v
```

```
mkdir terraform

cd terraform
```

---

## 📁 Structure

```
Trend/

├── dist/

├── Dockerfile

├── nginx.conf

├── README.md

└── terraform/

    └── main.tf
```
Write main.tf (Terraform Script)

1)VPC → Network for EC2 & EKS

2)Subnet → Private & public

3)Internet Gateway → Allow public access

4)Security Group → Open ports 22, 3000, 80

5)EC2 Instance → For Jenkins

6)EKS Cluster → Kubernetes Cluster

7)Node Group → EC2 workers for EKS
---

## 📝 Terraform main.tf

```
(Your FULL code unchanged 👇)

provider "aws" {

region = "ap-south-1"

}

# ---------------- VPC ----------------

resource "aws_vpc" "trend_vpc" {

cidr_block = "10.0.0.0/16"

tags = {

Name = "trend-vpc"

}

}

# ---------------- Subnet 1 ----------------

resource "aws_subnet" "trend_subnet" {

vpc_id = aws_vpc.trend_vpc.id

cidr_block = "10.0.1.0/24"

availability_zone = "ap-south-1a"

map_public_ip_on_launch = true

tags = {

Name = "trend-subnet-1"

}

}

# ---------------- Subnet 2 ----------------

resource "aws_subnet" "trend_subnet2" {

vpc_id = aws_vpc.trend_vpc.id

cidr_block = "10.0.2.0/24"

availability_zone = "ap-south-1b"

map_public_ip_on_launch = true

tags = {

Name = "trend-subnet-2"

}

}

# ---------------- Internet Gateway ----------------

resource "aws_internet_gateway" "trend_igw" {

vpc_id = aws_vpc.trend_vpc.id

tags = {

Name = "trend-igw"

}

}

# ---------------- Route Table ----------------

resource "aws_route_table" "trend_rt" {

vpc_id = aws_vpc.trend_vpc.id

route {

cidr_block = "0.0.0.0/0"

gateway_id = aws_internet_gateway.trend_igw.id

}

tags = {

Name = "trend-route-table"

}

}

# ---------------- Route Table Association ----------------

resource "aws_route_table_association" "subnet1_association" {

subnet_id = aws_subnet.trend_subnet.id

route_table_id = aws_route_table.trend_rt.id

}

resource "aws_route_table_association" "subnet2_association" {

subnet_id = aws_subnet.trend_subnet2.id

route_table_id = aws_route_table.trend_rt.id

}

# ---------------- Security Group ----------------

resource "aws_security_group" "trend_sg" {

name = "trend-sg"

description = "Allow SSH and Web"

vpc_id = aws_vpc.trend_vpc.id

ingress {

from_port = 22

to_port = 22

protocol = "tcp"

cidr_blocks = ["0.0.0.0/0"]

}

ingress {

from_port = 3000

to_port = 3000

protocol = "tcp"

cidr_blocks = ["0.0.0.0/0"]

}

ingress {

from_port = 80

to_port = 80

protocol = "tcp"

cidr_blocks = ["0.0.0.0/0"]

}

ingress {

from_port = 8080

to_port = 8080

protocol = "tcp"

cidr_blocks = ["0.0.0.0/0"]

}

egress {

from_port = 0

to_port = 0

protocol = "-1"

cidr_blocks = ["0.0.0.0/0"]

}

tags = {

Name = "trend-sg"

}

}

# ---------------- Jenkins EC2 ----------------

resource "aws_instance" "jenkins" {

ami = "ami-019715e0d74f695be"

instance_type = "c7i-flex.large"

subnet_id = aws_subnet.trend_subnet.id

key_name = "ApplicationServer-keypair"

vpc_security_group_ids = [aws_security_group.trend_sg.id]

user_data = <<-EOF

#!/bin/bash

apt update -y

apt install docker.io -y

systemctl start docker

systemctl enable docker

usermod -aG docker ubuntu

apt install openjdk-17-jdk -y

apt install unzip -y

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86\_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

./aws/install

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x kubectl

mv kubectl /usr/local/bin/

apt update -y

apt install jenkins -y

systemctl start jenkins

systemctl enable jenkins

EOF

tags = {

Name = "jenkins-server"

}

}

# ---------------- EKS Cluster ----------------

module "eks" {

source = "terraform-aws-modules/eks/aws"

version = "~> 20.0"

cluster_name = "trend-eks-cluster"

cluster_version = "1.29"

vpc_id = aws_vpc.trend_vpc.id

subnet_ids = [

aws_subnet.trend_subnet.id,

aws_subnet.trend_subnet2.id

]

eks_managed_node_groups = {

workers = {

instance_types = ["t3.small"]

min_size = 1

max_size = 3

desired_size = 2

}

}

}
```

---

## 🚀 Run Terraform

```
terraform init -upgrade

terraform validate

terraform plan

terraform apply
```
--
## 🌐 Networking

* VPC
* Subnet
* Internet Gateway
* Security
* Security Group (22, 80, 3000 open)

---

## 💻 Compute

* EC2 instance → Jenkins server
* and install dependencies like java

---

## ☸️ Kubernetes

* EKS cluster
* 2 worker nodes

---

## 🧱 Architecture Structure

```id="arch01"
VPC
│
├── Subnet
│    Subnet 1 (ap-south-1a)
│    Subnet 2 (ap-south-1b)
│
├── Internet Gateway
│
├── Security Group
│
├── EC2 Instance
│      Jenkins Server
│
└── EKS Cluster
       ├── Worker Node 1
       └── Worker Node 2
```

---

# 🚀 JENKINS SETUP

---

## 🌐 Open Jenkins

```id="jenk01"
http://EC2-PUBLIC-IP:8080
```

---

## 🔑 Get Password

```id="jenk02"
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

## 🔌 Connect SSH to Jenkins EC2
And these are the dependencies to install Jenkins i added these to terraform file .I am refering these for refernce

---

## 📦 Jenkins Dependencies (Reference)

Install Jenkins:
[https://www.jenkins.io/doc/book/installing/linux/](https://www.jenkins.io/doc/book/installing/linux/)

---

## ☕ Installation of Java

```id="java01"
sudo apt update

sudo apt install fontconfig openjdk-21-jre

java -version
```

---

## 📦 Install Jenkins (LTS)

```id="jenk03"
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \

 https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \

 https://pkg.jenkins.io/debian-stable binary/ | sudo tee \

 /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update

sudo apt install Jenkins
```

---

## 🐳 Install Docker on Jenkins Server

```id="dock01"
sudo apt install docker.io -y

sudo systemctl start docker

sudo systemctl enable docker

docker --version
```

---

## 🔐 Give Permissions

```id="perm01"
sudo usermod -aG docker jenkins

sudo systemctl restart Jenkins
```

---

## 🔄 Restart Jenkins

```id="jenk04"
sudo systemctl restart jenkins
```

---

## ☁️ AWS CLI Installation

```id="aws01"
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

sudo apt install unzip -y

unzip awscliv2.zip

sudo ./aws/install

aws --version
```

---

# 🔐 IAM CONFIGURATION

---

## IAM Principal

* terraform-user
* Type: Standard
* Group: system:masters

---

## Access Policy

* AdministratorAccess
* AmazonEKSClusterPolicy

---

## Access Scope

* Cluster

---

## EKS Access Entry

* IAM principal: terraform-user
* Type: Standard
* Access policy: AmazonEKSClusterAdminPolicy
* Access scope: Cluster

---

## 🌐 Networking Settings

* Change endpoint to **public and private**

---

# 🔗 CONNECT JENKINS TO EKS

---

## Configure AWS Credentials

```id="aws02"
aws configure
```

Enter:

* AWS Access Key ID
* AWS Secret Access Key
* Region: ap-south-1
* Output format: json

---

## Connect to EKS

```id="eks01"
aws eks update-kubeconfig --region ap-south-1 --name trend-eks-cluster
```

---

## ✅ Verify Cluster

```id="eks02"
kubectl get nodes
```

```id="eks03"
kubectl get pods
```

```id="eks04"
kubectl get deployment
```

---

# ⚠️ ERROR HANDLING

---

## ❌ Error: Large File Size Exceeded

```id="err01"
File terraform-provider-aws_v6.36.0_x5.exe is 812.61 MB
```

---

## 📌 Cause

Terraform downloaded provider binaries into `.terraform` folder.

---

## ✅ Solution

Add `.gitignore` rules.

---

# 📄 .gitignore

```id="git01"
# Terraform
terraform/.terraform/
terraform/*.tfstate
terraform/*.tfstate.backup
terraform/.terraform.lock.hcl

# Node
node_modules/

# Env
.env

# OS
.DS_Store
```

---

# 📄 .dockerignore

```id="dock02"
.git
.gitignore
node_modules
README.md
```

---

# 🚀 GIT COMMANDS

```id="git02"
git init 

git add .

git commit -m "Initial commit"

git branch -M main

git remote add origin https://github.com/Lakshmanhari/trend-app.git

git push -u origin main
```

---

# 📁 FINAL PROJECT STRUCTURE

```id="struct01"
Trend

├── dist
├── k8s
│    ├── deployment.yaml
│    └── service.yaml
├── terraform
│    └── main.tf
├── Dockerfile
├── nginx.conf
├── .gitignore
└── .dockerignore
```

---

# 🚀 CI/CD PIPELINE

---

## 🌐 Jenkins URL

```id="pipe01"
http://<JENKINS-IP>:8080
```

---

## 🔌 Install Plugins

* Git
* Docker
* Pipeline
* Kubernetes
* GitHub Integration

---

## 🔁 Webhook Setup

```id="web01"
http://<JENKINS-IP>:8080/github-webhook/
```

---

## ⚙️ Create Pipeline Job

* New Item → Pipeline
* Pipeline script from SCM

---

## 📜 Jenkinsfile

```id="jenkfile01"
pipeline {
    agent any

    stages {

        // ---------------- Clone Repo ----------------
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/Lakshmanhari/trend-app.git'
            }
        }

        // ---------------- Build Docker Image ----------------
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t lakshmanhari/trend-app:latest .'
            }
        }

        // ---------------- Push to DockerHub ----------------
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push lakshmanhari/trend-app:latest
                    '''
                }
            }
        }

        // ---------------- Deploy to Kubernetes ----------------
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-eks-creds',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        echo "Updating kubeconfig and deploying app..."
                        aws eks update-kubeconfig --region ap-south-1 --name trend-eks-cluster
                       
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                         
                        
                        kubectl rollout restart deployment/trend-app
                        kubectl rollout status deployment/trend-app
                    '''
                }
            }
        }

    } // end of stages
    post {
        always {
            echo 'Cleaning up Docker login...'
            sh 'docker logout'
        }
        success {
            echo 'Pipeline completed successfully! Your new changes are live.'
        }
        failure {
            echo 'Pipeline failed. Check the logs above for errors.'
        }
    }
} // end of pipeline
```

---

# 🔐 JENKINS CREDENTIALS

---

## DockerHub Credentials

* ID: dockerhub-creds
* Username: Docker Hub username
* Password: Docker Hub password

---

## AWS Credentials

* ID: aws-eks-creds
* Username: AWS Access Key
* Password: AWS Secret Key

---

# 📊 MONITORING (PROMETHEUS + GRAFANA)

---

## 🟢 Step 1 — Install Helm

```id="mon01"
helm version
```

---

## 🟢 Step 2 — Add Repo

```id="mon02"
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update
```
👉 This downloads Prometheus packages


---

## 🟢 Step 3 — Install Stack

```id="mon03"
helm install prometheus prometheus-community/kube-prometheus-stack \

--namespace monitoring --create-namespace
```
👉 This installs:
Prometheus (data collector)
Grafana (dashboard)


---

## 🟢 Step 4 — Check Pods

```id="mon04"
kubectl get pods -n monitoring
```
👉 Make sure all are:
Running


---

## 🟢 Step 5 — Get Grafana Password

```id="mon05"
kubectl get secret -n monitoring prometheus-grafana \

-o jsonpath="{.data.admin-password}" | base64 -d
```

👉 This gives login password

---

## 🟢 Step 6 — Expose Grafana

```id="mon06"
kubectl edit svc prometheus-grafana -n monitoring
```

Change:

```id="mon07"
type: ClusterIP
```

To:

```id="mon08"
type: LoadBalancer
```

---

## 🟢 Step 7 — Get URL

```id="mon09"
kubectl get svc -n monitoring
```
👉 Open:
EXTERNAL-IP
👉 Login:
Username: admin
Password: (from step 5)

---

## 🟢 Step 8 — Dashboards

Import dashboards
Inside Grafana:

Go to Dashboards

Open Kubernetes dashboards
* ✅ 1. Kubernetes / Compute Resources / Cluster
* ✅ 2. Kubernetes / Compute Resources / Node (Pods)
* ✅ 3. Kubernetes / Compute Resources / Pod
* ✅ 4. Kubernetes / Networking / Cluster

---

## 🟢 Step 9 — Label Service

```id="mon10"
kubectl label svc trend-service app=trend-app
```

---

## 🟢 Step 10 — Create ServiceMonitor

```id="mon11"
nano trend-monitor.yaml
```

```id="mon12"
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: trend-app-monitor
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: trend-app
  namespaceSelector:
    matchNames:
      - default
  endpoints:
  - port: http
    path: /health
    interval: 15s

```

---

## 🟢 Step 11 — Apply

```id="mon13"
kubectl apply -f trend-monitor.yaml
```

---

## 🟢 Step 12 — Verify

```id="mon14"

In Grafana → Explore:
up
```

---

# 🧱 FINAL FLOW

```id="flow01"
Developer (Git Push)
        │
        ▼
     GitHub
        │
        ▼
   CodePipeline
        │
        ▼
    CodeBuild
        │
        ▼
     Docker Build
        │
        ▼
   Amazon ECR
        │
        ▼
   Amazon EKS
        │
        ▼
 Kubernetes Pods
        │
        ▼
 AWS LoadBalancer
        │
        ▼
     End Users
```

---

## ✅ DONE

---




