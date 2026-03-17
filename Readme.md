

**🚀 TREND APPLICATION DEPLOYMENT (DEVOPS PROJECT)**

**📁 Project Structure**

**Trend**

&#x20;**├─ README.md**

&#x20;**└─ dist**

&#x20;     **├─ index.html**

&#x20;     **├─ assets**

&#x20;     **└─ vite.svg**

📌 Understanding the Application



What you saw:



dist/

&#x20;index.html

&#x20;assets/



These are static website files.



So instead of NodeJS we will use a web server like Nginx to serve them.



**🚀 DAY 1 — DOCKER + DOCKERHUB**

**🚀 Step 1 — Clone the Repository**



Open terminal in VS Code or PowerShell:



**git clone https://github.com/Vennilavanguvi/Trend.git**

**cd Trend**



**📁 Create Dockerfile**

Where



Inside the Trend project root folder:



Trend

&#x20;├─ dist

&#x20;└─ Dockerfile

Tool



Docker



Why we use Docker



To package the application with its runtime so it can run anywhere.



**📝 Create Dockerfile**



Open terminal:



🚀  Dockerfile



**notepad Dockerfile**





**FROM nginx:latest**



**COPY nginx.conf /etc/nginx/conf.d/default.conf**



**COPY dist/ /usr/share/nginx/html**



**EXPOSE 80**



**CMD \["nginx", "-g", "daemon off;"]**



\------------------------------

&#x20;Understand the Dockerfile

\-------------------------------

Line by line explanation.



Base Image

FROM nginx:latest



Use official Nginx image from DockerHub.



Copy Nginx Config

COPY nginx.conf /etc/nginx/conf.d/default.conf



Replace default nginx config with our config.



Copy Application

COPY dist/ /usr/share/nginx/html



Move website files into nginx web directory.



Expose Port

EXPOSE 3000



Tell Docker that the container will run on port 3000.



Start Nginx

CMD \["nginx", "-g", "daemon off;"]



Start nginx when container runs.

**-------------------------**

**🚀 Step 3 —  Build Docker Image**



Now build the image.



**docker build -t trend-app .**



output:



Successfully built

Successfully tagged trend-app



Check image:



**docker images**



You should see:



trend-app





**🚀 Step 4 — Run Docker Container**



**docker run -d -p 3000:80 react-deploy-app**



Explanation

Option	Meaning

\-d	run container in background

\-p 3000:80	map port 3000 to container port 80





**🌐 Step 5 — Open Application**



Open in browser:



http://localhost:3000



**📊 Step 6 — Verify Container**



**docker ps**



**docker images**

**----------------------------**

**🚀 Step 7 — Push Image to DockerHub**

**Login**



**docker login**



Tag Image



**docker tag react-deploy-app lakshmanhari/trend-app**



Push



**docker push lakshmanhari/trend-app**



Test Pull



**docker pull lakshmanhari/trend-app**







📌 DevOps Flow

React App (dist)

&#x20;       ↓

Docker Image

&#x20;       ↓

Docker Hub

📁 Correct Folder Structure

Trend/

├── dist/

├── nginx.conf

├── Dockerfile

└── README.md



**------------------------------------**

**📝 Create nginx.conf**

Since this is a static frontend app, we will use Nginx web server.





**notepad nginx.conf**



server {

&#x20;   listen 80;



&#x20;   location / {

&#x20;       root /usr/share/nginx/html;

&#x20;       index index.html index.htm;

&#x20;       try\_files $uri $uri/ /index.html;

&#x20;   }

}



Why we do this



This file tells Nginx:



Run server on port 3000

Serve files from /usr/share/nginx/html

If route not found → return index.html

\---------------



**Explanation**



dist/



This contains the production React build (HTML, CSS, JS).



Nginx will serve these files.



nginx.conf



Configuration file for the Nginx web server.



Tells Nginx to run on port 3000 and serve the dist files.



Dockerfile



Instructions for Docker to build the container image.



\------------------------------------------

**🚀 DAY 2 — TERRAFORM + AWS + EKS**





**Step 1 — Login to DockerHub**



In PowerShell run:



**docker login**



It will ask:



**Username:**

**Password:**



Enter your DockerHub username and password.



If login is successful you will see something like:



Login Succeeded

\-------------

**Check Your Local Image**



Run:



docker images



You should see:



trend-app



output



REPOSITORY   TAG       IMAGE ID

trend-app    latest    7d8f9ab23

**----------------------------------------------**

**Verify on DockerHub**



Go to:



https://hub.docker.com



Open Repositories.



You should see:



**trend-app**

\----------------------------------------------



**Terraform setup** 



**📦 Install Terraform**



**terraform -v**



📁 Create Terraform Folder



**mkdir terraform**

**cd terraform**



📁 Structure

Trend/

├── dist/

├── Dockerfile

├── nginx.conf

├── README.md

└── terraform/

&#x20;     └── main.tf



**Write main.tf (Terraform Script)**



1)VPC → Network for EC2 \& EKS



2)Subnet → Private \& public



3)Internet Gateway → Allow public access



4)Security Group → Open ports 22, 3000, 80



5)EC2 Instance → For Jenkins



6)EKS Cluster → Kubernetes Cluster



7)Node Group → EC2 workers for EKS





**📝 Terraform main.tf**



(Your FULL code unchanged 👇)



provider "aws" {

&#x20; region = "ap-south-1"

}



\# ---------------- VPC ----------------

resource "aws\_vpc" "trend\_vpc" {

&#x20; cidr\_block = "10.0.0.0/16"



&#x20; tags = {

&#x20;   Name = "trend-vpc"

&#x20; }

}



\# ---------------- Subnet 1 ----------------

resource "aws\_subnet" "trend\_subnet" {

&#x20; vpc\_id                  = aws\_vpc.trend\_vpc.id

&#x20; cidr\_block              = "10.0.1.0/24"

&#x20; availability\_zone       = "ap-south-1a"

&#x20; map\_public\_ip\_on\_launch = true



&#x20; tags = {

&#x20;   Name = "trend-subnet-1"

&#x20; }

}



\# ---------------- Subnet 2 ----------------

resource "aws\_subnet" "trend\_subnet2" {

&#x20; vpc\_id                  = aws\_vpc.trend\_vpc.id

&#x20; cidr\_block              = "10.0.2.0/24"

&#x20; availability\_zone       = "ap-south-1b"

&#x20; map\_public\_ip\_on\_launch = true



&#x20; tags = {

&#x20;   Name = "trend-subnet-2"

&#x20; }

}



\# ---------------- Internet Gateway ----------------

resource "aws\_internet\_gateway" "trend\_igw" {

&#x20; vpc\_id = aws\_vpc.trend\_vpc.id



&#x20; tags = {

&#x20;   Name = "trend-igw"

&#x20; }

}



\# ---------------- Route Table ----------------

resource "aws\_route\_table" "trend\_rt" {

&#x20; vpc\_id = aws\_vpc.trend\_vpc.id



&#x20; route {

&#x20;   cidr\_block = "0.0.0.0/0"

&#x20;   gateway\_id = aws\_internet\_gateway.trend\_igw.id

&#x20; }



&#x20; tags = {

&#x20;   Name = "trend-route-table"

&#x20; }

}



\# ---------------- Route Table Association ----------------

resource "aws\_route\_table\_association" "subnet1\_association" {

&#x20; subnet\_id      = aws\_subnet.trend\_subnet.id

&#x20; route\_table\_id = aws\_route\_table.trend\_rt.id

}



resource "aws\_route\_table\_association" "subnet2\_association" {

&#x20; subnet\_id      = aws\_subnet.trend\_subnet2.id

&#x20; route\_table\_id = aws\_route\_table.trend\_rt.id

}



\# ---------------- Security Group ----------------

resource "aws\_security\_group" "trend\_sg" {

&#x20; name        = "trend-sg"

&#x20; description = "Allow SSH and Web"

&#x20; vpc\_id      = aws\_vpc.trend\_vpc.id



&#x20; ingress {

&#x20;   from\_port   = 22

&#x20;   to\_port     = 22

&#x20;   protocol    = "tcp"

&#x20;   cidr\_blocks = \["0.0.0.0/0"]

&#x20; }



&#x20; ingress {

&#x20;   from\_port   = 3000

&#x20;   to\_port     = 3000

&#x20;   protocol    = "tcp"

&#x20;   cidr\_blocks = \["0.0.0.0/0"]

&#x20; }



&#x20; ingress {

&#x20;   from\_port   = 80

&#x20;   to\_port     = 80

&#x20;   protocol    = "tcp"

&#x20;   cidr\_blocks = \["0.0.0.0/0"]

&#x20; }



&#x20; ingress {

&#x20;   from\_port   = 8080

&#x20;   to\_port     = 8080

&#x20;   protocol    = "tcp"

&#x20;   cidr\_blocks = \["0.0.0.0/0"]

&#x20; }



&#x20; egress {

&#x20;   from\_port   = 0

&#x20;   to\_port     = 0

&#x20;   protocol    = "-1"

&#x20;   cidr\_blocks = \["0.0.0.0/0"]

&#x20; }



&#x20; tags = {

&#x20;   Name = "trend-sg"

&#x20; }

}



\# ---------------- Jenkins EC2 ----------------

resource "aws\_instance" "jenkins" {



&#x20; ami           = "ami-019715e0d74f695be"

&#x20; instance\_type = "c7i-flex.large"

&#x20; subnet\_id     = aws\_subnet.trend\_subnet.id

&#x20; key\_name      = "ApplicationServer-keypair"



&#x20; vpc\_security\_group\_ids = \[aws\_security\_group.trend\_sg.id]



&#x20; user\_data = <<-EOF

&#x20; #!/bin/bash

&#x20; apt update -y

&#x20; apt install docker.io -y

&#x20; systemctl start docker

&#x20; systemctl enable docker

&#x20; usermod -aG docker ubuntu

&#x20; apt install openjdk-17-jdk -y

&#x20; apt install unzip -y

&#x20; curl "https://awscli.amazonaws.com/awscli-exe-linux-x86\_64.zip" -o "awscliv2.zip"

&#x20; unzip awscliv2.zip

&#x20; ./aws/install

&#x20; curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

&#x20; chmod +x kubectl

&#x20; mv kubectl /usr/local/bin/

&#x20; apt update -y

&#x20; apt install jenkins -y

&#x20; systemctl start jenkins

&#x20; systemctl enable jenkins

&#x20; EOF



&#x20; tags = {

&#x20;   Name = "jenkins-server"

&#x20; }

}



\# ---------------- EKS Cluster ----------------

module "eks" {

&#x20; source  = "terraform-aws-modules/eks/aws"

&#x20; version = "\~> 20.0"



&#x20; cluster\_name    = "trend-eks-cluster"

&#x20; cluster\_version = "1.29"



&#x20; vpc\_id = aws\_vpc.trend\_vpc.id



&#x20; subnet\_ids = \[

&#x20;   aws\_subnet.trend\_subnet.id,

&#x20;   aws\_subnet.trend\_subnet2.id

&#x20; ]



&#x20; eks\_managed\_node\_groups = {

&#x20;   workers = {

&#x20;     instance\_types = \["t3.small"]

&#x20;     min\_size     = 1

&#x20;     max\_size     = 3

&#x20;     desired\_size = 2

&#x20;   }

&#x20; }

}

**🚀 Run Terraform**



**terraform init -upgrade**

**terraform validate**

**terraform plan**

**terraform apply**

\------------------------------------------

**Terraform will create:**



* Networking
* VPC
* Subnet
* Internet Gateway
* Security
* Security Group (22, 80, 3000 open)



**Compute**

* EC2 instance → Jenkins server

&#x20;and install dependencies like java

* Kubernetes



* EKS cluster

&#x20;  2 worker nodes





**VPC**

**│**

**├── Subnet**

**|    Subnet 1 (ap-south-1a)**

**|    Subnet 2 (ap-south-1b)**

**│**

**├── Internet Gateway**

**│**

**├── Security Group**

**│**

**├── EC2 Instance**

**│      Jenkins Server**

**│**

**└── EKS Cluster**

&#x20;      **├── Worker Node 1**

&#x20;      **└── Worker Node 2**





**🚀 JENKINS SETUP**

**🌐 Open Jenkins**



http://EC2-PUBLIC-IP:8080



🔑 Get Password



sudo cat /var/lib/jenkins/secrets/initialAdminPassword



Connect ssh to Jenkins Ec2

\----------------------------------------------------------------------------------------

And these are the dependencies to install Jenkins i added these to terraform file .I am refering these for refernce



Install Jenkins

(refer these Jenkins documentation for installing)

https://www.jenkins.io/doc/book/installing/linux/



Installation of Java

sudo apt update

sudo apt install fontconfig openjdk-21-jre

java -version



Long Term Support release



sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \\

&#x20; https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb \[signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \\

&#x20; https://pkg.jenkins.io/debian-stable binary/ | sudo tee \\

&#x20; /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update

sudo apt install Jenkins



nstall Docker on Jenkins Server



Run these commands inside your EC2 terminal.



Install Docker:



sudo apt install docker.io -y



Start Docker:



sudo systemctl start docker



Enable Docker at boot:



sudo systemctl enable docker



Check Docker:



docker --version



\-------------------------------------------------------------------------------------

**🔐 Give Permissions**



sudo usermod -aG docker jenkins

sudo systemctl restart Jenkins



Restart Jenkins:



sudo systemctl restart jenkins



Now Jenkins can run Docker builds.



Step 1 — Download AWS CLI

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86\_64.zip" -o "awscliv2.zip"

\--

Step 2 — Install unzip

sudo apt install unzip -y

\---

Step 3 — Unzip AWS CLI

unzip awscliv2.zip

\---

Step 4 — Install AWS CLI

sudo ./aws/install

\---

Step 5 — Verify installation

aws --version





\---------------------------------------

IAM principal

terraform-user



Type

Standard



Username

Leave default.



Group

system:masters



Access policy

AdministratorAccess

AmazonEKSClusterPolicy



Access scope

Cluster

\--------------------

Then click Create.



**In EKS Create access entry and add:**



**IAM principal: terraform-user**



**Type: Standard**



**Access policy: AmazonEKSClusterAdminPolicy**



**Access scope: Cluster**



and  in networking -->Manage--> change the end point to puplic and private

\--------------------------------

**Configure AWS credentials**



Because your Jenkins server must access Amazon EKS.



Run:



**aws configure**



Enter:



AWS Access Key ID

AWS Secret Access Key

Region: ap-south-1

Output format: json



Connect your Jenkins server to the EKS cluster you created with Terraform.



Command:



**aws eks update-kubeconfig --region ap-south-1 --name trend-eks-cluster**



Then test:



**kubectl get nodes**



If it shows worker nodes, your cluster is connected.

**kubectl get nodes**



NAME                                        STATUS   ROLES    AGE   VERSION

ip-10-0-1-210.ap-south-1.compute.internal   Ready    <none>   51m   v1.29.15-eks-ecaa3a6

ip-10-0-2-31.ap-south-1.compute.internal    Ready    <none>   51m   v1.29.15-eks-ecaa3a6



**kubectl get pods**



NAME                         READY   STATUS    RESTARTS   AGE

trend-app-554b65d744-fxrnt   1/1     Running   0          16m

trend-app-554b65d744-vj8qq   1/1     Running   0          16m



**kubectl get deployment**



NAME        READY   UP-TO-DATE   AVAILABLE   AGE

trend-app   2/2     2            2           16m

now my Jenkins Ec2 server is connected EKS

\--------------------------------------

**1️⃣ Error: Large File Size Exceeded**



Error message:



File terraform-provider-aws\_v6.36.0\_x5.exe is 812.61 MB

this exceeds GitHub's file size limit of 100 MB



Cause:



When you ran terraform init, Terraform downloaded provider binaries into the .terraform folder.



Examples:



terraform-provider-aws.exe

terraform-provider-kubernetes.exe



These files were hundreds of MB, and GitHub allows only 100MB maximum per file.



Solution:



We added rules in .gitignore to prevent these files from being pushed.



Example:



terraform/.terraform/

terraform/\*.tfstate

terraform/\*.tfstate.backup





**✅Create Correct .gitignore**



Inside your Trend folder, run:



**notepad .gitignore**



\# Terraform

terraform/.terraform/

terraform/\*.tfstate

terraform/\*.tfstate.backup

terraform/.terraform.lock.hcl



\# Node

node\_modules/



\# Env

.env



\# OS

.DS\_Store



\----------------------------------------

**✅Create .dockerignore**



**Now run:**



**notepad .dockerignore**





.git

.gitignore

node\_modules

README.md



**Git Commands to pull the files in the Github**

git init 

git add .

git commit -m "Initial commit"

git branch -M main

git remote add origin https://github.com/Lakshmanhari/trend-app.git

git push -u origin main



**📁 Final Correct Project Structure**

**Your project should look like this:**





Trend

&#x20;├── dist

&#x20;├── k8s

&#x20;│    ├── deployment.yaml

&#x20;│    └── service.yaml

&#x20;├── terraform

&#x20;│    └── main.tf

&#x20;├── Dockerfile

&#x20;├── nginx.conf

&#x20;├── .gitignore

&#x20;└── .dockerignore



**🚀 CI/CD PIPELINE**

&#x20;**http://<JENKINS-IP>:8080**



I**nstall Plugins**



**In Jenkins → Manage Plugins, you installed:**



Git



Docker



Pipeline



Kubernetes



GitHub Integration



👉 These plugins allow Jenkins to:



Pull code



Build Docker image



Push to DockerHub



Deploy to Kubernetes



\--------------------------------------------

**Setup Webhook (IMPORTANT)**



**👉 In GitHub:**



**Go to Settings → Webhooks**



Add:



http://<JENKINS-IP>:8080/github-webhook/



✔ Now:

👉 Every git push → Jenkins pipeline runs automatically

\--------------------------------------------------



**Create Pipeline Job**



**In Jenkins:**



New Item → Pipeline



Add your GitHub repo



Select:



Pipeline script from SCM



\------------------------------------------

**Jenkinsfile**



pipeline {

&#x20;   agent any



&#x20;   stages {



&#x20;       // ---------------- Clone Repo ----------------

&#x20;       stage('Clone Repo') {

&#x20;           steps {

&#x20;               git branch: 'main', url: 'https://github.com/Lakshmanhari/trend-app.git'

&#x20;           }

&#x20;       }



&#x20;       // ---------------- Build Docker Image ----------------

&#x20;       stage('Build Docker Image') {

&#x20;           steps {

&#x20;               sh 'docker build -t lakshmanhari/trend-app:latest .'

&#x20;           }

&#x20;       }



&#x20;       // ---------------- Push to DockerHub ----------------

&#x20;       stage('Push to DockerHub') {

&#x20;           steps {

&#x20;               withCredentials(\[usernamePassword(

&#x20;                   credentialsId: 'dockerhub-credentials',

&#x20;                   usernameVariable: 'DOCKER\_USER',

&#x20;                   passwordVariable: 'DOCKER\_PASS'

&#x20;               )]) {

&#x20;                   sh '''

&#x20;                       echo $DOCKER\_PASS | docker login -u $DOCKER\_USER --password-stdin

&#x20;                       docker push lakshmanhari/trend-app:latest

&#x20;                   '''

&#x20;               }

&#x20;           }

&#x20;       }



&#x20;       // ---------------- Deploy to Kubernetes ----------------

&#x20;       stage('Deploy to Kubernetes') {

&#x20;           steps {

&#x20;               withCredentials(\[usernamePassword(

&#x20;                   credentialsId: 'aws-eks-creds',

&#x20;                   usernameVariable: 'AWS\_ACCESS\_KEY\_ID',

&#x20;                   passwordVariable: 'AWS\_SECRET\_ACCESS\_KEY'

&#x20;               )]) {

&#x20;                   sh '''

&#x20;                       echo "Updating kubeconfig and deploying app..."

&#x20;                       aws eks update-kubeconfig --region ap-south-1 --name trend-eks-cluster

&#x20;                      

&#x20;                       kubectl apply -f k8s/deployment.yaml

&#x20;                       kubectl apply -f k8s/service.yaml

&#x20;                        

&#x20;                       

&#x20;                       kubectl rollout restart deployment/trend-app

&#x20;                       kubectl rollout status deployment/trend-app

&#x20;                   '''

&#x20;               }

&#x20;           }

&#x20;       }



&#x20;   } // end of stages

&#x20;   post {

&#x20;       always {

&#x20;           echo 'Cleaning up Docker login...'

&#x20;           sh 'docker logout'

&#x20;       }

&#x20;       success {

&#x20;           echo 'Pipeline completed successfully! Your new changes are live.'

&#x20;       }

&#x20;       failure {

&#x20;           echo 'Pipeline failed. Check the logs above for errors.'

&#x20;       }

&#x20;   }

} // end of pipeline



**------------------------------------------------**

**IAM User permission**



AmazonEKSClusterPolicy



AmazonEKSWorkerNodePolicy



IAMFullAccess



AmazonEC2FullAccess



Add AWS Credentials in Jenkins



Go to Jenkins Dashboard → Manage Jenkins → Credentials → Global → Add Credentials



**1️⃣ Docker Hub credentials (dockerhub-creds)**



Select: Username with password



Username: Your Docker Hub username (lakshmanhari)



Password: Your Docker Hub password (or access token if you use 2FA)



ID: dockerhub-creds (or any name you choose)



**2️⃣ AWS credentials for EKS (aws-eks-creds)**



Jenkins doesn’t show an AWS credentials type here, but you can still use Username with password as a workaround:



Username: AWS Access Key ID



Password: AWS Secret Access Key



ID: aws-eks-creds



\----------------------------------------------------------------------------------



**🎯 MONITORING – COMPLETE STEPS (SIMPLE)**

**🟢 STEP 1: Install Helm**



👉 (Tool to install Prometheus easily)



helm version



✔ Confirms Helm is installed



🟢 STEP 2: Add Prometheus repo

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update



👉 This downloads Prometheus packages

**-----------------------------**

**🟢 STEP 3: Install Prometheus + Grafana**

helm install prometheus prometheus-community/kube-prometheus-stack \\

\--namespace monitoring --create-namespace



👉 This installs:



Prometheus (data collector)



Grafana (dashboard)



Alertmanager

\-------------------

🟢 STEP 4: Check pods

kubectl get pods -n monitoring



👉 Make sure all are:



Running

\---------------

**🟢 STEP 5: Get Grafana password**

kubectl get secret -n monitoring prometheus-grafana \\

\-o jsonpath="{.data.admin-password}" | base64 -d



👉 This gives login password



**----------------------------**

**🟢 STEP 6: Expose Grafana**

kubectl edit svc prometheus-grafana -n monitoring



👉 Change:



type: ClusterIP



➡ to:



type: LoadBalancer

**---------------------**

**🟢 STEP 7: Get URL**

kubectl get svc -n monitoring



👉 Open:



EXTERNAL-IP



👉 Login:



Username: admin



Password: (from step 5)



\----------------



**🟢 STEP 8: Import dashboards**



Inside Grafana:



Go to Dashboards



Open Kubernetes dashboards



👉 Now you can see:

✅ 1. Kubernetes / Compute Resources / Cluster



✅ 2. Kubernetes / Compute Resources / Node (Pods)



✅ 3. Kubernetes / Compute Resources / Pod



✅ 4. Kubernetes / Networking / Cluster

\---------------------------------------------------

🟢 STEP 9: Connect YOUR APP



👉 Add label:



kubectl label svc trend-service app=trend-app



\----------------------------------------------------

**🟢 STEP 10: Create ServiceMonitor**

**nano trend-monitor.yaml**



Paste:



apiVersion: monitoring.coreos.com/v1

kind: ServiceMonitor

metadata:

&#x20; name: trend-app-monitor

&#x20; namespace: monitoring

spec:

&#x20; selector:

&#x20;   matchLabels:

&#x20;     app: trend-app

&#x20; namespaceSelector:

&#x20;   matchNames:

&#x20;     - default

&#x20; endpoints:

&#x20; - port: http

&#x20;   path: /

&#x20;   interval: 15s

🟢 STEP 11: Apply it

kubectl apply -f trend-monitor.yaml



👉 Now Prometheus starts monitoring your app

\----------------------------------------------------

**🟢 STEP 12: Verify**



In Grafana → Explore:



up





