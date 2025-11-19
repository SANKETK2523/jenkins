COMPLETE JENKINS CI/CD GUIDE (AWS EC2 + Ubuntu + GitHub + Apache)

Includes: Commands, configuration, explanations → Everything you need.

✅ 1) Launch EC2 Instance
1.1 Create Instance

Go to AWS EC2 → Launch Instance

Name: jenkins-cicd-server

AMI: Ubuntu 22.04

Instance type: t2.micro

Key pair: Create or use existing

Security Group (VERY IMPORTANT):

22 → SSH

8080 → Jenkins

80 → Apache Web Server

Source: Anywhere (for testing)

1.2 Connect to EC2
ssh -i your-key.pem ubuntu@EC2_PUBLIC_IP

✅ 2) Install Dependencies on EC2

We install:

Java

Jenkins

Apache

Git

2.1 Update server
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
3.1 Open Jenkins

Visit:

http://EC2_PUBLIC_IP:8080

3.2 Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

3.3 Complete setup

Install Suggested Plugins

Create admin user

✅ 4) Setup Web Directory for Deployment

We will deploy website files to:

/var/www/html

Give Jenkins permission:
sudo chown -R jenkins:jenkins /var/www/html
sudo chmod -R 755 /var/www/html

✅ 5) Install Jenkins Plugin (Publish Over SSH)
In Jenkins UI:

Go to: Manage Jenkins → Plugins → Available

Search: Publish Over SSH

Install + restart Jenkins

Configure SSH (allows Jenkins to copy files)

In Jenkins:

Manage Jenkins → Configure System → Publish over SSH

Add a new SSH Server:

Name: EC2

Hostname: 127.0.0.1

Username: jenkins

Remote Directory: /var/www/html

Click Test → should pass.

✅ 6) Create a GitHub Repo with HTML Page
6.1 Create GitHub repository

Name: my-website

6.2 Create HTML file

index.html:

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

Commands:

git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USER/my-website.git
git push -u origin main

✅ 7) Create Jenkins Job (Using Publish Over SSH)
7.1 Create Freestyle Job

Jenkins Dashboard → New Item

Name: deploy-my-website

Select: Freestyle project

7.2 Source Code Management

Select Git

Repository URL:
https://github.com/YOUR_USER/my-website.git

Branch: main

7.3 Build Step (Deploy)

Add:
Send files or execute commands over SSH

Configure:

SSH Server: select EC2

Source files:

*


Remove prefix: (leave blank)

Remote directory:

/var/www/html

7.4 Execute Command After Transfer

In the same section → "Exec command":

sudo systemctl restart apache2


Click Save

✅ 8) Configure GitHub Webhook

In GitHub:

Repo → Settings → Webhooks → Add Webhook

Use:

http://EC2_PUBLIC_IP:8080/github-webhook/


Content type: application/json

Event: Just the push event

Click Add Webhook

Webhook should show ✓ green if successful.

✅ 9) Fix Apache to Show Your Site

Make sure Apache serves /var/www/html

Check Apache site config
sudo nano /etc/apache2/sites-available/000-default.conf


Ensure:

DocumentRoot /var/www/html


Restart Apache:

sudo systemctl restart apache2

Open your site:
http://EC2_PUBLIC_IP/

✅ 10) Test the Pipeline
Step 1: Make a change

Modify index.html:

<h1>New site deployed by Jenkins!</h1>

Step 2: Push code
git add .
git commit -m "updated website"
git push

Step 3: GitHub webhook triggers Jenkins

Jenkins:

pulls repo

transfers files via SSH

restarts Apache

site updates automatically

Step 4: View updated website
http://EC2_PUBLIC_IP/


🎉 Your CI/CD pipeline works!
