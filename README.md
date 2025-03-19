# **🚀 Secure Cloud with Tailscale, Caddy & Nextcloud AIO**

This setup enables **secure remote access** to your **Nextcloud AIO instance** hosted on your **home server**, using **Tailscale for networking**, **Caddy as a reverse proxy**, and **Cloudflare** for **DNS & security**.

---

## **📚 Table of Contents**
1. [Directory Structure](#📁-directory-structure)
2. [Prerequisites](#📋-prerequisites)
3. [Key Components](#🔧-key-components)
4. [Workflow & Traffic Flow](#🔄-workflow--traffic-flow)
5. [Uploading Large Files](#📂-uploading-large-files)
6. [Security Enhancements](#🔒-security-enhancements)
7. [Deployment & Configuration](#🛠️-deployment--configuration)
8. [Installing Tailscale](#🛠️-installing-tailscale)
9. [Installing Nextcloud AIO](#🛠️-installing-nextcloud-aio)
10. [Troubleshooting](#🛠️-troubleshooting)
11. [Hardware Used](#💻-hardware-used-personal-setup)
12. [License](#📜-license)
13. [Summary](#✅-summary)

---

## **📁 Directory Structure**
The project is organized as follows:

```
/NextcloudDeploy
├── HomeServer
│   └── compose.yml          # Defines Nextcloud AIO services for the home server
├── EC2
│   ├── compose.yml          # Defines Caddy reverse proxy services for AWS EC2
│   ├── Caddyfile            # Configures Caddy reverse proxy
│   └── Caddy.Dockerfile     # Dockerfile for building Caddy with Cloudflare DNS plugin
├── LICENSE                  # Apache License 2.0
└── README.md                # This file
```

---

## **📋 Prerequisites**
Before starting, ensure you have the following:
- **Docker** and **Docker Compose** installed on both your home server and AWS EC2 instance.
- A **Cloudflare account** with a domain name configured.
- A **Tailscale account** for secure networking.
- Basic knowledge of **DNS management** and **Docker commands**.

---

## **🔧 Key Components**
### **1️⃣ Cloudflare (DNS & Security)**
- **Cloudflare is used for DNS management** and protecting your domain.
- An **A record** in Cloudflare points your **domain to your AWS EC2 instance**.
- **Proxied mode (orange cloud) is enabled** for added security.
- **Cloudflare handles SSL/TLS termination and firewall protection**.

### **2️⃣ AWS EC2 (Cloud Server)**
- Runs **Caddy inside Docker** to **reverse proxy** traffic to your home server.
- **Tailscale is installed on the host machine** to securely tunnel traffic.
- **Cloudflare protects the EC2 instance from direct public access**.

### **3️⃣ Home Server (Nextcloud AIO & Tailscale)**
- **[Nextcloud AIO](https://github.com/nextcloud/all-in-one)** is used to provide a **self-hosted cloud storage solution**.
- **Tailscale runs inside Docker**, connecting securely to AWS EC2.
- No direct public exposure—**all traffic routes through Tailscale**.

---

## **🔄 Workflow & Traffic Flow**

Below is a visual representation of the workflow:

```
+-------------------+       +-------------------+       +-------------------+
|   User Browser    |       |    Cloudflare     |       |      AWS EC2      |
|  yourdomain.com   | ----> |   DNS & Security  | ----> |   Caddy Reverse   |
|                   |       |                   |       |      Proxy        |
+-------------------+       +-------------------+       +-------------------+
                                                             |
                                                             v
                                                   +-------------------+
                                                   |    Home Server    |
                                                   | Nextcloud AIO via |
                                                   |    Tailscale      |
                                                   +-------------------+
```

1. **User accesses `yourdomain.com` via Cloudflare**.
2. **Cloudflare forwards requests to AWS EC2 (Caddy running in Docker)**.
3. **Caddy proxies traffic to the Tailscale node running on the home server**.
4. **Tailscale securely forwards the request to Nextcloud AIO (port 11000)**.
5. **Nextcloud AIO serves the request and sends responses back via the same path**.

---

## **💡 Use Case: No Port Forwarding Required**
Many ISPs **do not allow port forwarding** or provide **dynamic IPs**, making self-hosting difficult.  
This architecture **eliminates the need for port forwarding** by using **Tailscale** as a secure private tunnel, allowing access to your **home server from anywhere** without exposing it to the internet.

---

## **📂 Uploading Large Files**
### **Cloudflare Proxy Mode Limitations**
- Even though **Nextcloud AIO uses 100MB chunking by default**, when using **Cloudflare's proxy mode**, uploads larger than **~3.5GB** often fail.
- **Upload speed is also significantly slower** due to Cloudflare’s processing overhead.

### **Workaround for Large Files**
To upload large files **without Cloudflare's limitations**, follow these steps:
1. **Temporarily disable Cloudflare proxy** (turn off the orange cloud in Cloudflare DNS settings).  
2. **Upload files to Nextcloud** using direct access through your domain.
3. **Re-enable Cloudflare proxy** after completing the upload.

### **Performance Improvement Observed**
- **By disabling Cloudflare proxy, uploads work reliably** even for very large files.
- **Observed upload speed is ~3MBps**, which is much faster than when using Cloudflare proxy.
- If anyone finds **further optimizations**, please **create a pull request**.
- If there are **security improvement suggestions**, feel free to **comment**.

---

## **🔒 Security Enhancements**
- **Only ports 443 (HTTPS) & 22 (SSH for management) are open on EC2**.
- **Cloudflare protects against DDoS and malicious requests**.
- **Tailscale prevents direct exposure of the home server to the internet**.
- **Cloudflare SSL ensures encrypted connections**.
- **Regular updates for Docker containers and system packages are recommended**.

---

## **🛠️ Deployment & Configuration**
1. **Set up Cloudflare DNS**:
   - Create an **A record** pointing your domain to EC2.
   - Enable **proxied mode** for security.

2. **Deploy Nextcloud AIO & Tailscale on the Home Server**:
   - Use the `compose.yml` in the `HomeServer` directory to start **Nextcloud AIO** & **Tailscale**.

3. **Deploy Caddy on AWS EC2**:
   - Use the `compose.yml` in the `EC2` directory to start **Caddy** with the Cloudflare DNS plugin.

4. **Secure Everything**:
   - Use **Cloudflare firewall rules** to block unwanted traffic.
   - Configure **Cloudflare SSL/TLS settings**.
   - Regularly **update all containers and packages**.

---

## **🛠️ Environment Variables**
Create a `.env` file in both the `HomeServer` and `EC2` directories with the following variables:

### **HomeServer .env**
```
NEXTCLOUD_PORT=8080
NEXTCLOUD_APACHE_PORT=11000
NEXTCLOUD_IP_BINDING=0.0.0.0
NEXTCLOUD_SKIP_DOMAIN_VALIDATION=true
NEXTCLOUD_TRUSTED_DOMAINS=nextcloud.example.com
NEXTCLOUD_DATADIR=/mnt/cloud/nextcloud1
NEXTCLOUD_MEMORY_LIMIT=4096M
NEXTCLOUD_UPLOAD_LIMIT=256G
NEXTCLOUD_OVERWRITEHOST=nextcloud.example.com
```

### **EC2 .env**
```
CLOUDFLARE_API_KEY=your-cloudflare-api-key
CADDY_HOSTNAME=yourdomain.com
HOMESERVER_IP=100.x.x.x  # Tailscale IP of the home server
HOMESERVER_PORT=11000
```

---

## **🛠️ Installing Tailscale**
### **1️⃣ Install Tailscale on AWS EC2**
1. SSH into your EC2 instance:
   ```bash
   ssh -i <your-key.pem> ec2-user@<ec2-public-ip>
   ```
2. Install Tailscale:
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   ```
3. Start Tailscale and authenticate:
   ```bash
   sudo tailscale up
   ```
4. Note the Tailscale IP of the EC2 instance from the Tailscale admin console.

### **2️⃣ Install Tailscale on Home Server**
1. SSH into your home server:
   ```bash
   ssh user@<home-server-ip>
   ```
2. Install Tailscale:
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   ```
3. Start Tailscale and authenticate:
   ```bash
   sudo tailscale up
   ```
4. Note the Tailscale IP of the home server from the Tailscale admin console.

---

## **🛠️ Installing Nextcloud AIO**
### **Accessing Nextcloud AIO via Local IP**
1. Ensure the `compose.yml` in the `HomeServer` directory is running:
   ```bash
   cd HomeServer
   docker-compose up -d
   ```
2. Open a browser and navigate to the local IP of your home server on port 8080:
   ```
   http://<home-server-local-ip>:8080
   ```
3. Follow the on-screen instructions to complete the Nextcloud AIO setup.

---

## **⚡ Quick Start**
Follow these steps to quickly deploy the setup:
1. **Clone the repository**:
   ```bash
   git clone https://github.com/your-repo/NextcloudDeploy.git
   cd NextcloudDeploy
   ```

2. **Set up environment variables**:
   - Create a `.env` file in the `EC2` directory and populate it with your **Cloudflare API key** and **Tailscale auth key**.

3. **Deploy Nextcloud AIO & Tailscale on your home server**:
   ```bash
   cd HomeServer
   docker-compose up -d
   ```

4. **Configure Cloudflare DNS**:
   - Add an **A record** pointing your domain to your AWS EC2 instance.
   - Enable **proxied mode** for added security.

5. **Deploy Caddy on AWS EC2**:
   ```bash
   cd EC2
   docker-compose up -d
   ```

6. **Access your Nextcloud instance**:
   - Open your browser and navigate to `https://yourdomain.com`.

---

## **🛠️ Troubleshooting**
### **Common Issues & Fixes**
1. **Cloudflare Proxy Mode Upload Limitations**:
   - Refer to the **Uploading Large Files** section for a workaround.

2. **Tailscale Connection Issues**:
   - Ensure Tailscale is running and authenticated on both the home server and AWS EC2 instance.
   - Check the Tailscale admin console for connection status.

3. **Caddy Reverse Proxy Not Working**:
   - Verify the `Caddyfile` configuration.
   - Check Docker logs for errors:
     ```bash
     docker logs <caddy-container-name>
     ```

4. **SSL/TLS Errors**:
   - Ensure Cloudflare SSL settings are set to **Full (strict)**.
   - Confirm that the domain has a valid SSL certificate.

5. **Slow Upload Speeds**:
   - Temporarily disable Cloudflare proxy mode as described in the **Uploading Large Files** section.

---

## **💻 Hardware Used (Personal Setup)**
I personally run this setup on a **Raspberry Pi 4B 8GB model** with a **5TB hard disk**, ensuring efficient performance and sufficient storage for my cloud.

---

## **📜 License**
This project is licensed under **Apache License 2.0**.

---

## **✅ Summary**
This setup ensures **maximum security, private cloud storage, and seamless remote access** using **Tailscale, Caddy, and Cloudflare**.  

It is **perfect for people who cannot port forward** and allows **temporary disabling of Cloudflare proxy for large file uploads**.


