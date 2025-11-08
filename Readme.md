
# 🌐 Piyush Dalvi – PORTFOLIO-APP-CI-CD

### 🚀 Overview
This project demonstrates **complete CI/CD automation** of a personal **Portfolio Website** using **Git, GitHub, Jenkins, AWS EC2, and Nginx**.  
The goal is to automatically host a static portfolio webpage on an EC2 instance whenever new code is pushed to GitHub.

---

## ⚡ Architecture Diagram

![Architecture Diagram](./imgs/Gemini_Generated_Image_a4734ha4734ha473.png)

---

## 🧩 Tech Stack

| Tool | Purpose |
|------|----------|
| **Git** | Version control system to track code changes |
| **GitHub** | Remote repository to store and manage project code |
| **Jenkins** | CI/CD automation tool for continuous integration and deployment |
| **AWS EC2 (Ubuntu)** | Cloud instance used as Jenkins server and portfolio hosting server |
| **Nginx** | Web server for hosting portfolio site |
| **SSH Keys** | Secure authentication between Jenkins and EC2 deployment server |

---

## ⚙️ Project Workflow – Step by Step Explanation

### 🧱 **1. Project Development (Local System)**
- Created a static **portfolio website** using HTML, CSS, and images.
- Verified it locally using VS Code Live Server.
- Folder structure:
  ```
  /PORTFOLIO-APP-CI-CD
  ├── index.html
  ├── css/
  ├── js/
  └── Jenkinsfile
  ```

---

### 🧰 **2. Initialize Git and Push Code to GitHub**
Used Git to version control and push code to GitHub.

```bash
git init
git add .
git commit -m "Initial portfolio upload"
git branch -M main
git remote add origin https://github.com/dalvipiyush07/PORTFOLIO-APP-CI-CD.git
git push -u origin main
```

✅ After this, the complete project (HTML, CSS, images, Jenkinsfile) is available publicly on GitHub.  
👉 [GitHub Repository](https://github.com/dalvipiyush07/PORTFOLIO-APP-CI-CD)

---

### ⚙️ **3. Setup Jenkins on Ubuntu EC2 (Server 1)**
1. Install Jenkins on Ubuntu EC2:
   ```bash
   sudo hostnamectl hostname jenkins
   sudo apt update
   sudo apt install openjdk-17-jdk -y
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb http://pkg.jenkins.io/debian/ stable main > /etc/apt/sources.list.d/jenkins.list'
   sudo apt update
   sudo apt install jenkins -y
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```
2. Access Jenkins via:
   👉 `http://<Jenkins-Server-Public-IP>:8080`

3. Install Plugins:
   - Git Plugin
   - SSH Agent Plugin

4. Add SSH credentials:
   - ID: `jenkinskey`
   - Username: `ubuntu`
   - Private Key: pasted from `.pem` file

---

### 🖥️ **4. Setup Portfolio EC2 (Server 2)**
1. Launch a new Ubuntu EC2 instance for portfolio hosting.
2. Open inbound port **80 (HTTP)** in AWS Security Group.
3. Jenkins pipeline will handle Nginx installation automatically.

---

### 🔁 **5. Jenkins Pipeline Configuration**
In Jenkins Dashboard:
- Create a new Pipeline Project → `portfolio-deploy`
- Select “Pipeline Script from SCM”
- SCM: Git
- Repo URL: `https://github.com/dalvipiyush07/PORTFOLIO-APP-CI-CD.git`
- Branch: `main`
- Script Path: `Jenkinsfile`

---

## 🧾 Jenkinsfile (CI/CD Pipeline Script)

```groovy
pipeline {
    agent any

    environment {
        DEPLOY_USER = 'ubuntu'
        DEPLOY_HOST = 'AWS_EC2_PUBLIC_IP'      // EC2 (Portfolio Server)
        DEPLOY_PATH = '/var/www/html'
        SSH_CREDENTIALS_ID = 'jenkinskey'      // Jenkins Credential ID
        REPO_URL = 'https://github.com/dalvipiyush07/PORTFOLIO-APP-CI-CD.git'
        BRANCH = 'main'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo '📦 Checking out portfolio code from GitHub...'
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Prepare Deployment Server') {
            steps {
                echo '⚙️ Installing Nginx and preparing deployment path...'
                sshagent (credentials: ["${SSH_CREDENTIALS_ID}"]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} "
                        sudo apt update -y &&
                        sudo apt install nginx -y &&
                        sudo systemctl enable nginx &&
                        sudo systemctl start nginx &&
                        sudo rm -rf ${DEPLOY_PATH}/*"
                    '''
                }
            }
        }

        stage('Deploy Portfolio') {
            steps {
                echo '🚀 Deploying portfolio to EC2...'
                sshagent (credentials: ["${SSH_CREDENTIALS_ID}"]) {
                    sh '''scp -o StrictHostKeyChecking=no -r * ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}/
                    ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} "
                        sudo chown -R www-data:www-data ${DEPLOY_PATH} &&
                        sudo systemctl restart nginx"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful! Portfolio live on Nginx server.'
        }
        failure {
            echo '❌ Deployment Failed. Check Jenkins logs for details.'
        }
    }
}
```

---

## 🌍 Live Deployment  
🚧 **Deployment ready on AWS EC2 – Coming soon!**  

---

## 📸 Screenshots  

| Screenshot | Description |
|-------------|-------------|
| ![1](./imgs/success.png) | Jenkins console output |
| ![2](./imgs/github.png) | GitHub repository view |
| ![3](./imgs/ec2.png) | EC2 instance dashboard |
| ![4](./imgs/vs.png) | VS Code structure |
| ![5](./imgs/port.png) | Jenkins build success screen |

---

## ✅ Key Highlights

- Fully automated CI/CD using Jenkins  
- Zero manual deployment — push → auto-update  
- Secure & scalable AWS setup  
- Separate Jenkins and Deployment EC2 servers  
- Nginx as production-grade static web host  

---

## 👨‍💻 Author

**Piyush Dalvi**  
DevOps & Cloud Engineer  
💼 [GitHub Profile](https://github.com/dalvipiyush07)

---

⭐ *If you found this project helpful, don’t forget to star the repo!*
