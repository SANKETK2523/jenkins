# ======================================================
# Jenkins CI/CD Setup on AWS EC2 (Ubuntu) with GitHub
# ======================================================

# -----------------------------
# Step 1: Launch EC2 Instance
# -----------------------------
# (Manual step in AWS Console)
# - Ubuntu Server 22.04 LTS
# - Security Group: SSH(22), HTTP(80), Jenkins(8080)
# - Download .pem key
# Connect via SSH:
# chmod 400 mykey.pem
# ssh -i mykey.pem ubuntu@<EC2_PUBLIC_IP>

# -----------------------------
# Step 2: Update & Install Dependencies
# -----------------------------
sudo apt update && sudo apt upgrade -y

# Install Java (required for Jenkins)
sudo apt install openjdk-11-jdk -y
java -version

# Install Apache web server
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2

# Install Git
sudo apt install git -y
git --version

# Install Jenkins
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

# Access Jenkins via: http://<EC2_PUBLIC_IP>:8080
# Get initial admin password:
# sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# -----------------------------
# Step 3: Configure Jenkins
# -----------------------------
# (Manual step in browser)
# - Complete setup wizard
# - Install suggested plugins
# - Create admin user

# -----------------------------
# Step 4: Setup Web Directory
# -----------------------------
sudo mkdir -p /var/www/html/myapp
sudo chown -R ubuntu:ubuntu /var/www/html/myapp

# -----------------------------
# Step 5: Jenkins Plugin (Publish Over SSH)
# -----------------------------
# (Manual step in Jenkins Dashboard)
# - Manage Jenkins → Manage Plugins → Install "Publish Over SSH"
# - Configure: Hostname=<EC2_PUBLIC_IP>, Username=ubuntu, Remote Directory=/var/www/html/myapp
# - Use private key (your .pem file or Jenkins key)

# -----------------------------
# Step 6: Create GitHub Repo & HTML
# -----------------------------
# (Manual: Create repo "myapp")
# Add HTML file:
cat <<EOF > index.html
<!DOCTYPE html>
<html>
<head>
    <title>My Jenkins Deployment</title>
</head>
<body>
    <h1>Hello from Jenkins CI/CD!</h1>
</body>
</html>
EOF

# Initialize git and push to GitHub
git init
git add index.html
git commit -m "Initial commit"
git branch -M main
git remote add origin <GITHUB_REPO_URL>
git push -u origin main

# -----------------------------
# Step 7: Create Jenkins Job
# -----------------------------
# (Manual in Jenkins)
# - New Item → Freestyle Project → "Deploy-MyApp"
# - Source Code Management → Git URL
# - Build Trigger → GitHub hook trigger
# - Build → Execute Shell:
echo "Deploying to Apache web server..."
cp -r * /var/www/html/myapp/

# -----------------------------
# Step 8: Configure GitHub Webhook
# -----------------------------
# (Manual in GitHub Settings)
# - Settings → Webhooks → Add webhook
# - Payload URL: http://<EC2_PUBLIC_IP>:8080/github-webhook/
# - Content type: application/json
# - Event: push

# -----------------------------
# Step 9: Configure Apache
# -----------------------------
sudo sed -i 's|DocumentRoot /var/www/html|DocumentRoot /var/www/html/myapp|g' /etc/apache2/sites-available/000-default.conf
sudo systemctl restart apache2

# -----------------------------
# Step 10: Test the Pipeline
# -----------------------------
# Make a change to trigger Jenkins:
echo "<p>Updated via Jenkins</p>" >> index.html
git add index.html
git commit -m "Update page"
git push origin main

# Visit http://<EC2_PUBLIC_IP>/ to see the deployed page
