# Domain configuration

## Only Http : 

- create a ec2 container and start your ec2 server.
- git clone your server code and start using pm2(if nodejs).
- expose your port from security in bound groups
- now attach an elastic ip (create and attach)
- now go to your dns provider and attach A-record (see the below img).
![alt text](image.png)
- now its time to attach this domain with your ec2
- install and setup ngnix using this command

```bash
 
    sudo apt update
    sudo apt install nginx -y
    sudo systemctl enable --now nginx


```

- Connect Nginx to your Node.js application
```bash
   sudo nano /etc/nginx/sites-available/myapp
```
- For Ubuntu, paste this configuration, replacing example.com with your actual domain:
```bash
    server {
        listen 80;
        server_name example.com www.example.com;

        location / {
            proxy_pass http://127.0.0.1:3000;

            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
```

- Enable the configuration and reload Nginx:
```bash
    sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/myapp
    sudo nginx -t
    sudo systemctl reload nginx

```
- Test your domain
```bash
   nslookup example.com
```


## -----------------------------------------------------------------------------------------------------------------------------------
# Enable HTTPS on EC2 Using Certbot

This guide explains how to enable HTTPS for:

**`todo-app.ritesh-18.in`**

The application runs on Node.js port `3000`, with Nginx acting as a reverse proxy on port `80`. Certbot will obtain a free TLS certificate from Let's Encrypt and configure Nginx to serve HTTPS.

## Request flow

```text
Browser
  https://todo-app.ritesh-18.in (port 443)
                 |
                 v
          Nginx + TLS certificate
                 |
                 v
       Node.js application (port 3000)
```

## Step 1: Verify HTTP and DNS

First, confirm the application works over HTTP:

- http://todo-app.ritesh-18.in

It should show your application, not the default Nginx welcome page.

On EC2, check that DNS resolves to your Elastic IP:

```bash
getent ahostsv4 todo-app.ritesh-18.in
```

The returned public IPv4 address should match your EC2 Elastic IP.

## Step 2: Allow HTTPS in the EC2 security group

1. Open the [EC2 Console](https://console.aws.amazon.com/ec2/).
2. Select **Instances → your instance → Security**.
3. Open the attached security group and choose **Edit inbound rules**.
4. Add this rule:

| Field | Value |
|---|---|
| Type | HTTPS |
| Protocol | TCP |
| Port | `443` |
| Source | `0.0.0.0/0` |

Keep the existing HTTP inbound rule on port `80`. Save the rules.

## Step 3: Install Certbot

Run these commands on your Ubuntu EC2 instance:

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

Verify installation:

```bash
certbot --version
```

## Step 4: Request and install the certificate

Run:

```bash
sudo certbot --nginx -d todo-app.ritesh-18.in
```

Follow the prompts:

1. Enter an email address you can access for certificate expiry notices.
2. Agree to the terms of service.
3. Choose whether to receive optional email communications.
4. If asked whether to redirect HTTP traffic to HTTPS, select the redirect option.

Certbot will request the certificate and update Nginx configuration automatically. Domain DNS must point to the correct public IP, and Let's Encrypt must be able to reach the server for validation.

## Step 5: Test Nginx and HTTPS

Run:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Then open:

- https://todo-app.ritesh-18.in

You can also test from the EC2 terminal:

```bash
curl -I https://todo-app.ritesh-18.in
```

A successful response confirms that the HTTPS endpoint is responding. An HTTP status such as `200`, `301`, or `302` may be normal depending on your application and redirect configuration.

## Step 6: Test automatic certificate renewal

Check the renewal timer:

```bash
systemctl status certbot.timer
```

Test the renewal process without replacing the live certificate:

```bash
sudo certbot renew --dry-run
```

A successful dry run indicates that renewal can complete.

## Troubleshooting

### Certbot validation times out

- Confirm the EC2 security group allows inbound TCP ports `80` and `443`.
- Confirm any host firewall allows these ports.
- Verify that the domain's A record points to the Elastic IP.
- Confirm Nginx is running and reachable over HTTP.

### DNS points to the wrong address

Run:

```bash
getent ahostsv4 todo-app.ritesh-18.in
```

Correct the DNS A record if the address does not match your Elastic IP. DNS changes may take time to propagate.

### Nginx configuration errors

Run:

```bash
sudo nginx -t
sudo certbot certificates
```

Fix any Nginx configuration error before retrying Certbot.

### Useful diagnostics

```bash
sudo systemctl status nginx --no-pager
sudo ss -lntp | grep -E ':80|:443'
sudo tail -n 50 /var/log/nginx/error.log
```

## Important notes

- Do not expose the Node.js port `3000` publicly unless you specifically need to. Nginx can proxy requests to `127.0.0.1:3000`.
- Keep the EC2 security group limited to the ports and sources you need.
- Do not delete your existing Nginx site configuration when Certbot modifies it.
- If Certbot reports an error, save the complete output so you can troubleshoot the specific failure.

