# AWS EC2 Guide: Accessing Server & Basic File Operations

This guide covers setting up an AWS Security Group, restricting `.pem` file permissions to **Read-Only** on both Linux and Windows, connecting to the EC2 server, and performing basic file CRUD operations.

---

## 1. AWS Security Group Configuration

To allow external traffic to access your web application or server, you must explicitly permit inbound connections in the [AWS Management Console](https://amazon.com "AWS Management Console").

### Required Inbound Rules
1. Navigate to the **EC2 Dashboard** -> **Instances** -> select your instance -> click the **Security** tab -> click your **Security Group**.
2. Click **Edit inbound rules** and add the following rules:

| Type | Protocol | Port Range | Source | Description |
| :--- | :--- | :--- | :--- | :--- |
| **SSH** | TCP | 22 | `My IP` or `0.0.0.0/0` | Secure shell access (Keep to your IP for production) |
| **HTTP** | TCP | 80 | `0.0.0.0/0` | Public web server traffic |
| **HTTPS** | TCP | 443 | `0.0.0.0/0` | Secure public web server traffic |
| **Custom TCP** | TCP | 3000 | `0.0.0.0/0` | Common node/web application port |

*Note: For public access, ensure the **Source** column is set to `0.0.0.0/0` (IPv4) or `::/0` (IPv6).*

---

## 2. Restricting `.pem` File Permissions (Read-Only)

SSH clients will reject your private key if the file permissions are too open. Below are the commands to safely restrict permissions to **Read-Only** based on your OS.

### On Linux / macOS (Terminal)
Run the standard `chmod 400` command to ensure the file is only readable by the owner:
```bash
chmod 400 /path/to/aws-access-key.pem
```

### On Windows (Command Prompt - `cmd`)
Windows relies on Access Control Lists (ACLs). You must clear the default inherited permissions and grant explicit read-only access strictly to your active user account:

```cmd
:: 1. Reset permissions to a clean slate
icacls "C:\Users\ritesh-18\Desktop\ritesh\AWS-credentials\pem-file\aws-access-key.pem" /reset

:: 2. Disable inheritance and remove inherited permissions
icacls "C:\Users\ritesh-18\Desktop\ritesh\AWS-credentials\pem-file\aws-access-key.pem" /inheritance:r

:: 3. Grant explicit read-only access to your specific user profile
icacls "C:\Users\ritesh-18\Desktop\ritesh\AWS-credentials\pem-file\aws-access-key.pem" /grant:r "%username%":R
```

---

## 3. Connecting to your Server

Use your Windows Command Prompt or Linux Terminal to log into your remote server instance. Substitute your correct public IP address below:

```cmd
ssh -i "C:\Users\ritesh-18\Desktop\ritesh\AWS-credentials\pem-file\aws-access-key.pem" ubuntu@13.203.223.45
```

### Default OS Usernames
If the system drops a `Permission denied (publickey)` error, verify you are connecting with the correct default server username for your OS instance type:
* **Ubuntu:** `ubuntu`
* **Amazon Linux 2 / 2023:** `ec2-user`
* **CentOS:** `centos`
* **Debian:** `admin`

---

## 4. File CRUD Operations in Linux (SSH Session)

Once you are securely authenticated and inside the Linux remote shell terminal, you can perform basic CRUD (Create, Read, Update, Delete) file operations:

### **C**reate a File
Generate a brand-new blank file or use text inputs to create it:
```bash
touch my-app.log
```
Or immediately direct text inside it:
```bash
echo "Server initialization successful" > my-app.log
```

### **R**ead a File
Output the exact file content straight into your current console frame:
```bash
cat my-app.log
```

### **U**pdate a File
Append fresh data to the bottom of the existing log file content:
```bash
echo "New incoming configuration updated" >> my-app.log
```
*Alternatively, modify the file interactively using the integrated nano editor:*
```bash
nano my-app.log
```

### **D**elete a File
Permanently erase the targeting file from the active folder environment:
```bash
rm my-app.log
```
