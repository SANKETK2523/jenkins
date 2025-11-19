# Jenkins CI/CD Pipeline with AWS EC2, GitHub, and Apache

This is a **beginner-friendly guide** to set up a CI/CD pipeline using Jenkins on an AWS EC2 Ubuntu instance, deploying a simple HTML page from GitHub to an Apache web server.

---

## Step 1: Launch EC2 Instance

1. Log in to **AWS Console → EC2 → Launch Instance**.
2. Choose **Ubuntu Server 22.04 LTS**.
3. Instance Type: **t2.micro** (free tier if applicable).
4. Configure security group to allow:
   - SSH: 22
   - HTTP: 80
   - Jenkins: 8080
5. Launch instance and download your `.pem` key file.

**Connect via SSH:**

```bash
chmod 400 mykey.pem
ssh -i mykey.pem ubuntu@<EC2_PUBLIC_IP>
