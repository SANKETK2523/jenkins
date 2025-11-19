give me guide to jenkins CI/CD WITH AWS EC2 and github using aws ec2(ubuntu),jenkins,github webhooks,apache web server in following steps and a beginner friendly guide :-1) Launch EC2 Instance,2) Install Dependencies on EC2, 3) Configure Jenkins, 4) Setup Web Directory for Deployment, 5) Install Jenkins Plugin (Publish Over SSH),6) Create a GitHub Repo with HTML Page,7) Create Jenkins Job,8) Configure GitHub Webhook, 9) Fix Apache to Show Your Site,10) Test the Pipeline give commands correctly and explain
Step 1: Launch EC2 Instance

Log in to your AWS Console → EC2 → Launch Instance.

Choose Ubuntu Server 22.04 LTS.

Instance Type: t2.micro (free tier if applicable).

Configure security group to allow:

SSH: 22

HTTP: 80

Jenkins: 8080

Launch instance and download your .pem key file.

Connect via SSH:

chmod 400 mykey.pem
ssh -i mykey.pem ubuntu@<EC2_PUBLIC_IP>


Replace <EC2_PUBLIC_IP> with your instance’s public IP.

Step 2: Install Dependencies on EC2

Update packages:

sudo apt update && sudo apt upgrade -y


Install Java (required for Jenkins):

sudo apt install openjdk-11-jdk -y
java -version


Install Apache web server:

sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2


Install Git:

sudo apt install git -y
git --version


Install Jenkins:

wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins


Access Jenkins: http://<EC2_PUBLIC_IP>:8080

Unlock Jenkins using:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Step 3: Configure Jenkins

Complete setup wizard.

Install suggested plugins.

Create an admin user.

Jenkins is ready to create jobs.

Step 4: Setup Web Directory for Deployment

Apache default root: /var/www/html

Make a folder for your app (optional):

sudo mkdir -p /var/www/html/myapp
sudo chown -R ubuntu:ubuntu /var/www/html/myapp


This ensures Jenkins (running as ubuntu or jenkins) can write files to this directory.

Step 5: Install Jenkins Plugin (Publish Over SSH)

Go to Jenkins Dashboard → Manage Jenkins → Manage Plugins → Available.

Search for "Publish Over SSH" and install.

Configure in Manage Jenkins → Configure System → Publish over SSH:

Name: EC2

Hostname: <EC2_PUBLIC_IP>

Username: ubuntu

Remote Directory: /var/www/html/myapp

Use private key (your .pem file or generate Jenkins SSH key)

Step 6: Create a GitHub Repo with HTML Page

Go to GitHub → New Repository → myapp.

Create an index.html file:

<!DOCTYPE html>
<html>
<head>
    <title>My Jenkins Deployment</title>
</head>
<body>
    <h1>Hello from Jenkins CI/CD!</h1>
</body>
</html>


Push to GitHub:

git init
git add index.html
git commit -m "Initial commit"
git branch -M main
git remote add origin <GITHUB_REPO_URL>
git push -u origin main

Step 7: Create Jenkins Job

Dashboard → New Item → Freestyle Project → Deploy-MyApp.

Source Code Management → Git → Repository URL.

Build Triggers → GitHub hook trigger for GITScm polling.

Build → Execute Shell:

echo "Deploying to Apache web server..."
cp -r * /var/www/html/myapp/


(or use Publish over SSH if deploying remotely)

Step 8: Configure GitHub Webhook

Go to your GitHub repo → Settings → Webhooks → Add webhook.

Payload URL: http://<EC2_PUBLIC_IP>:8080/github-webhook/

Content type: application/json

Events: Just the push event.

Save webhook.

Now, each push triggers Jenkins to deploy automatically.

Step 9: Fix Apache to Show Your Site

Make sure Apache points to your directory:

sudo nano /etc/apache2/sites-available/000-default.conf


Set:

DocumentRoot /var/www/html/myapp


Restart Apache:

sudo systemctl restart apache2

Step 10: Test the Pipeline

Push a change to GitHub:

echo "<p>Updated via Jenkins</p>" >> index.html
git add index.html
git commit -m "Update page"
git push origin main


Jenkins should trigger the job automatically.

Visit http://<EC2_PUBLIC_IP>/ → You should see the updated page.

✅ Congratulations! You now have a simple CI/CD pipeline with:
