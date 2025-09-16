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

## ⚙️ Quick Start (Console)
1. Sign in to the AWS Console → **EC2 → Launch Instance**.  
2. Configure:
   - **Name:** `Custom-Apache-Web-Server`  
   - **AMI:** Amazon Linux 2023  
   - **Instance Type:** `t2.micro` (Free Tier eligible)  
   - **Key Pair:** choose as needed (you can use EC2 Instance Connect if not using an SSH key)  
   - **Networking:** Enable **Auto-assign Public IP**  
   - **Security Group (Inbound):**  
     - **SSH (22)** — Source: *your IP*  
     - **HTTP (80)** — Source: `0.0.0.0/0` (allow IPv4 web traffic)  
   - **User Data:** paste the provided user-data script (below)

3. Launch instance, then open `http://<PUBLIC_IPV4>` in your browser.

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

