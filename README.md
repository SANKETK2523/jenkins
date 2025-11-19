Jenkins CI/CD Pipeline — AWS EC2 (Ubuntu) + GitHub Webhooks + Apache Web Server

This guide walks you through setting up a complete CI/CD pipeline using:

AWS EC2 (Ubuntu)

Jenkins

GitHub + Webhooks

Apache Web Server

Publish Over SSH Plugin

✅ 1) Launch EC2 Instance
1.1 Create EC2 Instance

AWS Console → EC2 → Launch Instance

AMI: Ubuntu 22.04

Instance Type: t2.micro

Key Pair: Create or select existing

Security Group:

22 (SSH)

8080 (Jenkins)

80 (Apache)

Launch

1.2 Connect to EC2
ssh -i your-key.pem ubuntu@EC2_PUBLIC_IP

✅ 2) Install Dependencies on EC2
2.1 Update System
sudo apt update && sudo apt upgrade -y

2.2 Install Java
sudo apt install openjdk-17-jdk -y

2.3 Install Jenkins
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y

2.4 Start Jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins

2.5 Install Apache Web Server
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2

✅ 3) Configure Jenkins
3.1 Access Jenkins

Visit:

http://EC2_PUBLIC_IP:8080

3.2 Retrieve Jenkins Admin Password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

3.3 Initial Setup

Install Suggested Plugins

Create Admin User

✅ 4) Setup Web Directory for Deployment

Jenkins will deploy files into:

/var/www/html


Give Jenkins permission:

sudo chown -R jenkins:jenkins /var/www/html
sudo chmod -R 755 /var/www/html

✅ 5) Install Jenkins Plugin (Publish Over SSH)
5.1 Install Plugin

In Jenkins UI:

Manage Jenkins → Plugins → Available
Search: Publish Over SSH → Install

5.2 Configure SSH

Go to:

Manage Jenkins → Configure System → Publish Over SSH

Add:

Name: EC2

Hostname: 127.0.0.1

Username: jenkins

Remote Directory: /var/www/html

Click Test → Should succeed.

✅ 6) Create a GitHub Repo with HTML Page
6.1 Create Repository

Create GitHub repo: my-website

6.2 Add HTML File

Create index.html:

<!DOCTYPE html>
<html>
<head>
    <title>My CI/CD Website</title>
</head>
<body>
    <h1>Hello from Jenkins CI/CD Pipeline!</h1>
</body>
</html>

6.3 Push to GitHub
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USER/my-website.git
git push -u origin main

✅ 7) Create Jenkins Job
7.1 Create Freestyle Job

Jenkins → New Item

Name: deploy-my-website

Type: Freestyle project

7.2 Source Code (Git)

Repository URL:
https://github.com/YOUR_USER/my-website.git

Branch: main

7.3 Add Build Step (Deploy via SSH)

Add:
Send files or execute commands over SSH

Settings:

Server: EC2

Source files:

*


Remote directory:

/var/www/html

7.4 Add Post-Transfer Command
sudo systemctl restart apache2


Save the job.

✅ 8) Configure GitHub Webhook

Go to:

GitHub Repo → Settings → Webhooks → Add Webhook

Webhook URL:

http://EC2_PUBLIC_IP:8080/github-webhook/


Content type: application/json

Event: Just the push event

Save → Should show a green ✓

✅ 9) Fix Apache to Show Your Site

Ensure Apache points to the correct directory:

Check Virtual Host
sudo nano /etc/apache2/sites-available/000-default.conf


Must include:

DocumentRoot /var/www/html


Restart Apache:

sudo systemctl restart apache2


Check:

http://EC2_PUBLIC_IP/

✅ 10) Test the Pipeline
10.1 Update HTML File

Modify index.html:

<h1>Updated via Jenkins Pipeline!</h1>

10.2 Push to GitHub
git add .
git commit -m "Update website"
git push

10.3 Pipeline Execution

GitHub Webhook → Jenkins builds → Deploys to Apache

10.4 View Result

Open:

http://EC2_PUBLIC_IP/


You should now see your updated website.

🎉 Pipeline Complete!

You now have:

✔ Automated GitHub → Jenkins → Apache deployment
✔ GitHub Webhooks triggering CI/CD
✔ Full control to expand into advanced pipeline
