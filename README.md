# 🚀 Deploy a Custom Apache Web Server on Amazon EC2

This repository demonstrates how to deploy a **custom Apache (httpd) web server** on an **Amazon Linux 2023 EC2 instance**, automatically configured at launch using **EC2 User Data**. It includes a sample custom webpage and troubleshooting notes from a real run-through.

---

## 📖 Project Overview
- **Cloud Provider:** AWS  
- **Service:** Amazon EC2  
- **OS:** Amazon Linux 2023  
- **Web Server:** Apache (httpd)  
- **Automation:** EC2 User Data (bash script)  
- **Deliverable:** Custom HTML webpage deployed automatically

---

## Architecture Overview
![EC2 Architecture](images/aws_ec2_apache_architecture.png)

---

## ⚙️ Quick Start (Console)
1. Sign in to the AWS Console → **EC2 → Launch Instance**.  
2. Configure:
   - **Name:** `Custom-Apache-Web-Server`  
   - **AMI:** Amazon Linux 2023  
   ![AMI Screenshot](images/instance-name.jpg)

   - **Instance Type:** `t2.micro` (Free Tier eligible)  
   ![Instance Type Screenshot](images/choose-instance-type.jpg)

   - **Key Pair:** choose as needed (you can use EC2 Instance Connect if not using an SSH key)  
   ![Choosing a key pair Screenshot](images/choose-a-key-pair.jpg)

   - **Networking:** Enable **Auto-assign Public IP**  
   ![Auto-assign Public IP Screenshot](images/enable-auto-assign-ip.jpg)

   - **Security Group (Inbound):**  
   ![SSH & HTTP rules setup Screenshot](images/ssh-http-rules.jpg)

     - **SSH (22)** — Source: *your IP*  
     - **HTTP (80)** — Source: `0.0.0.0/0` (allow IPv4 web traffic)  

   - **User Data:** paste the provided user-data script (below)
   ![User Data Screenshot](images/webpage-user-data.jpg)

   - **Launch Instance:**
   ![Launched Instance Screenshot](images/launched-instance.jpg)

3. Launch instance, then open `http://<PUBLIC_IPV4>` in your browser.
![Webpage Screenshot](images/webpage.jpg)

---

## 🧾 User Data Script (paste into the User Data field)
```bash
#!/bin/bash
# Update package list and install Apache
dnf update -y
dnf install -y httpd

# Start and enable Apache service
systemctl start httpd
systemctl enable httpd

# Set permissions for Apache root directory
chown -R ec2-user:ec2-user /var/www/html

# Create a custom index.html page
cat <<'EOF' > /var/www/html/index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome to My Custom Apache Server</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            text-align: center;
            padding: 50px;
        }
        .container {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
            display: inline-block;
        }
        h1 { color: #333; }
        p  { color: #666; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Welcome to My Custom Apache Web Server!</h1>
        <p>Hosted on an Amazon Linux 2023 EC2 Instance.</p>
        <p>This page was deployed automatically using AWS User Data.</p>
    </div>
</body>
</html>
EOF

# Restart Apache to apply changes
systemctl restart httpd
```

---

## ✅ Verify the Web Server
1. In the **EC2 Console → Instances**, select your instance.  
2. Copy the **Public IPv4 address**.  
3. Open in your browser:  
```bash
http://<PUBLIC_IPV4>
```
4. You should see the custom webpage 🎉  

---

## 🛠 Troubleshooting — Real Issue Encountered & Resolution

### 🔎 Symptom Observed
- Browser could not reach the page.  
- Running `systemctl status httpd` returned:  

 Unit httpd.service could not be found.


### ⚡ Root Causes Identified
- Security Group initially allowed only **SSH (port 22)** (no HTTP rule).  
- The **EC2 User Data** did not run correctly at boot, so Apache was never installed.  

### 📝 Step-by-Step Fix
1. **Add HTTP inbound rule (Console)**  
 - EC2 → Instances → Select instance → **Security** → click **security group name** → **Edit inbound rules** → Add rule:  
   - **Type:** HTTP  
   - **Protocol:** TCP  
   - **Port range:** 80  
   - **Source:** `0.0.0.0/0`  
 - Save rules.  

 *(CLI alternative — requires AWS CLI configured)*:  
 ```bash
 aws ec2 authorize-security-group-ingress \
   --group-id sg-0123456789abcdef0 \
   --protocol tcp \
   --port 80 \
   --cidr 0.0.0.0/0
```

2.  **SSH into the instance (or use EC2 Instance Connect if no key pair)**  
```bash
ssh ec2-user@<PUBLIC_IPV4>
```

3. **Check Apache (httpd) status**
```bash
systemctl status httpd
```


If you see:

Unit httpd.service could not be found.


→ Apache isn’t installed.

4. **Install Apache manually**
```bash
sudo dnf update -y
sudo dnf install -y httpd
```

5. **Start and enable Apache**
```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

6. **Create or restore the custom index page(Minimal test page)**
```bash
sudo tee /var/www/html/index.html <<'HTML'
<h1>Hello from Apache on EC2 (Amazon Linux 2023)</h1>
HTML
```

Or reapply the full HTML from the User Data script (recommended).

7. **Test locally on the instance**
```bash
curl http://localhost
```

Expect to see the HTML output.

8. **Test from your browser**
```bash
http://<PUBLIC_IPV4>
```

9. **(Optional) Re-run user data script if automation failed**

Check for script saved by cloud-init:
```bash
sudo ls -l /var/lib/cloud/instance/scripts/
```

If you find user-data.txt (or similar), re-run:
```bash
sudo bash /var/lib/cloud/instance/scripts/user-data.txt
```

10. **Check cloud-init logs (for debugging)**
```bash
sudo tail -n 200 /var/log/cloud-init-output.log
sudo tail -n 200 /var/log/cloud-init.log
```

---

## ❓ Why This Happened

User Data can fail if:

- A package repo update fails.  
- The instance boots before networking is ready.  
- `cloud-init` logs show an error.  
- The missing `httpd` unit meant the install step did not complete.  

---

## 🔍 Additional Troubleshooting Tips

- **Check cloud-init logs:**
```bash
sudo cat /var/log/cloud-init-output.log
```
If using IPv6, add ::/0 for HTTP (optional). 0.0.0.0/0 is sufficient for IPv4.

View Apache logs if service fails to start:
```bash
sudo journalctl -u httpd -n 100
```
Consider Elastic IP for a stable IP.

Use IAM roles if you automate with CLI.

---

## 🎯 Key Learnings

- Always verify **Security Group inbound rules** (HTTP is required for web traffic).  
- `systemctl` output — “Unit ... could not be found” usually means the package isn’t installed.  
- **Cloud-init / User Data** is powerful but always check its logs to confirm it ran successfully.  

---

## 🔮 Next Steps

- Add **HTTPS** (ACM + Load Balancer or Nginx + Certbot).  
- Provision with **Terraform or CloudFormation** for reproducible infra.  
- Add **CI/CD** to deploy content changes automatically.  

---

## 👩‍💻 Authors

- **Belinda Ntinyari** – [GitHub](https://github.com/Bel-94) | [LinkedIn](https://www.linkedin.com/in/belinda-ntinyari/)  

---

## 📜 License

This project is licensed under the [MIT License](https://opensource.org/licenses/MIT).  
