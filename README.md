# my-static-website2
🚀 CI/CD Pipeline for a Static Website Using Jenkins, GitHub, Docker & Azure VM
This guide will walk you through setting up a CI/CD pipeline using Jenkins on an Ubuntu VM in Azure to deploy a static website using Docker.
________________________________________
🎯 Step-by-Step Plan
1.	Set up an Ubuntu VM in Azure
2.	Install Jenkins on the VM
3.	Install Docker on the VM
4.	Create a GitHub repository for the website
5.	Write a Dockerfile to containerize the website
6.	Write a Jenkinsfile to automate deployment
7.	Configure Jenkins to trigger builds from GitHub
8.	Deploy and verify the website
________________________________________
🛠️ Step 1: Set Up an Ubuntu VM in Azure
1.	Go to Azure Portal → Virtual Machines → Create VM.
2.	Choose:
o	Image: Ubuntu 20.04 LTS  
o	Size: Standard B1s (or larger for production)  
o	Authentication: SSH or password  
o	Allow HTTP (80) & SSH (22) in Networking  
3.	Click Create and wait for the VM to start.
4.	Get the public IP of your VM from the Azure portal.
________________________________________
🛠️ Step 2: Install Jenkins on the Ubuntu VM  
1️⃣ SSH into Your VM
```
ssh azureuser@<your-vm-ip>  
```
2️⃣ Install Java (Jenkins Dependency)  
```
sudo apt update && sudo apt install -y openjdk-11-jdk
```
3️⃣ Install Jenkins
```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
4️⃣ Start and Enable Jenkins
```
sudo systemctl start jenkins
sudo systemctl enable jenkins
```
5️⃣ Open Jenkins in Browser

•	Open http://<your-vm-ip>:8080
•	Get the initial admin password:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
•	Install recommended plugins and create an admin user.
________________________________________
🛠️ Step 3: Install Docker on the Ubuntu VM
```
sudo apt update
sudo apt install -y docker.io
```
Allow Jenkins to use Docker:
```
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```
Verify Docker installation:
```
docker --version
```
________________________________________
🛠️ Step 4: Create a GitHub Repository for the Website
1.	Go to GitHub → Create New Repository (e.g., static-website).
2.	Inside the repo, create an index.html:
```
<!DOCTYPE html>
<html>
<head><title>My Static Website</title></head>
<body><h1>Version 1.0</h1></body>
</html>
```
3.	Push the code to GitHub:
__________
🛠️ Step 5: Create a Dockerfile  
Inside the GitHub repo, create a file named Dockerfile:
```
FROM nginx:alpine
COPY . /usr/share/nginx/html
```
This:
•	Uses a lightweight NGINX image.  
•	Copies the website files into the default web server directory.  
Push the Dockerfile to GitHub
________________________________________
🛠️ Step 6: Write a Jenkinsfile for the CI/CD Pipeline  
Inside your GitHub repo, create a file named Jenkinsfile:
```
pipeline {
    agent any
    environment {
        IMAGE_NAME = 'static-website'
        CONTAINER_NAME = 'static-website-container'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    credentialsId: 'github-ssh-key', 
                    url: 'https://github.com/yourusername/static-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Stop Old Container') {
            steps {
                sh '''
                    if [ $(docker ps -q -f name=$CONTAINER_NAME) ]; then
                        docker stop $CONTAINER_NAME
                    fi
                    if [ $(docker ps -aq -f name=$CONTAINER_NAME) ]; then
                        docker rm $CONTAINER_NAME
                    fi
                '''
            }
        }

        stage('Run New Container') {
            steps {
                sh 'docker run -d --name $CONTAINER_NAME -p 80:80 $IMAGE_NAME'
            }
        }
    }
}
```
Push the Jenkinsfile:

________________________________________
🛠️ Step 7: Configure Jenkins to Trigger on GitHub Push
1.	Install GitHub Plugin in Jenkins
o	Go to Manage Jenkins → Plugins → Install GitHub Integration Plugin.  
2.	Set Up Jenkins Job  
o	Create a new pipeline job.  
o	Under "Pipeline", choose Pipeline script from SCM.  
o	Enter your GitHub repo URL.  
o	Save and trigger a build.  
3.	Set Up GitHub Webhook    
o	Go to GitHub Repo → Settings → Webhooks → Add Webhook.  
o	Payload URL: http://<your-jenkins-ip>:8080/github-webhook/  
o	Content Type: application/json  
o	Trigger: Just the push event.  
o	Save.  
Now, every time you push code to GitHub, Jenkins will automatically build and deploy the website.
________________________________________
🛠️ Step 8: Deploy and Verify the Website
1.	Open Jenkins and run the pipeline job.
2.	Check if the container is running:
3.	docker ps
Output should show an NGINX container running.
4.	Open your browser and visit:
5.	http://<your-vm-ip>
You should see the static website! 🎉
________________________________________
🚀 Summary  
Step |	What We Did  
Step 1	Created an Ubuntu VM in Azure  
Step 2	Installed Jenkins  
Step 3	Installed Docker  
Step 4	Set up a GitHub repo for the website  
Step 5	Created a Dockerfile to containerize the website  
Step 6	Wrote a Jenkinsfile for CI/CD  
Step 7	Configured Jenkins GitHub Webhook  
Step 8	Deployed and verified the website  
________________________________________
🎯 Next Steps  
•	Add NGINX as a Reverse Proxy for multiple services.  
•	Enable HTTPS with Let’s Encrypt.  
•	Push Docker images to Docker Hub and pull them on multiple VMs.  
Need help with advanced features? Let me know! 🚀🔥

