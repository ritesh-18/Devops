# Ubuntu EC2 NGINX Reverse Proxy Setup Guide

This guide details how to install and configure **NGINX** as a **reverse proxy** on an **AWS EC2 Ubuntu** instance to route external traffic to your backend application.

---

## Step 1: AWS Security Groups Configuration
Before executing any server commands, ensure your EC2 instance accepts web traffic. Edit the **Inbound Rules** of your EC2 Security Group:

* **HTTP**: Port `80` | Source: `0.0.0.0/0` (Anywhere)
* **HTTPS**: Port `443` | Source: `0.0.0.0/0` (Anywhere)
* **SSH**: Port `22` | Source: Your IP Address

---

## Step 2: Install NGINX on Ubuntu
SSH into your instance and run the following commands:

```bash
# Update the package index
sudo apt update

# Install NGINX
sudo apt install nginx -y

# Start and enable NGINX to run on system boot
sudo systemctl start nginx
sudo systemctl enable nginx
```

---

## Step 3: Configure the Reverse Proxy
Create a standalone configuration file for your app inside the `sites-available` directory:

1. Open the file editor:
   ```bash
   sudo nano /etc/nginx/sites-available/my-app.conf
   ```

2. Paste the configuration block below (adjust the `proxy_pass` port to match your internal application server):
   ```nginx
   server {
       listen 80;
       server_name your-domain.com www.your-domain.com; # Use _ if you don't have a domain yet

       location / {
           proxy_pass http://127.0.0.1:3000; # The local port where your backend runs
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

3. Enable the new configuration by creating a symbolic link and removing the default NGINX template:
   ```bash
   # Link the config to make it active
   sudo ln -s /etc/nginx/sites-available/my-app.conf /etc/nginx/sites-enabled/

   # Remove the default catch-all site configuration
   sudo rm /etc/nginx/sites-enabled/default
   ```

---

## Step 4: Validate and Restart NGINX
Test your syntax rules to ensure there are no configuration typos:

```bash
sudo nginx -t
```
*If it reports success, restart the service to apply your updates:*

```bash
sudo systemctl restart nginx
```

---

## (Optional) Step 5: Secure with Let's Encrypt SSL (HTTPS)
If you pointed a valid domain name to your instance, you can secure it with a free SSL certificate:

```bash
# Install Certbot and its NGINX plugin
sudo apt install certbot python3-certbot-nginx -y

# Obtain and automatically apply the SSL certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
```

