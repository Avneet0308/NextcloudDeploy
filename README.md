# **🚀 Secure Cloud with Tailscale, Caddy & Nextcloud AIO**  

This setup enables **secure remote access** to your **Nextcloud AIO instance** hosted on your **home server**, using **Tailscale for networking**, **Caddy as a reverse proxy**, and **Cloudflare** for **DNS & security**.

---

## **📁 Directory Structure**
The project is organized as follows:

```
/secure-cloud
├── docker-compose.yml       # Defines services (Nextcloud AIO, Tailscale, Caddy)
├── Caddyfile                # Configures Caddy reverse proxy
├── .env                     # Stores sensitive environment variables (Cloudflare API key, Tailscale auth key)
├── security.md              # Security best practices
├── README.md                # This file
└── LICENSE                  # Apache License 2.0 (recommended)
```

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
   - Run Docker Compose to start **Nextcloud AIO** & **Tailscale**.

3. **Deploy Caddy on AWS EC2**:
   - **[Caddy](https://caddyserver.com/)** inside Docker will **reverse proxy to the Tailscale node**.

4. **Secure Everything**:
   - Use **Cloudflare firewall rules** to block unwanted traffic.
   - Configure **Cloudflare SSL/TLS settings**.
   - Regularly **update all containers and packages**.

---

## **💻 Hardware Used (Personal Setup)**
I personally run this setup on a **Raspberry Pi 4B 8GB model** with a **5TB hard disk**, ensuring efficient performance and sufficient storage for my cloud.

---

## **📜 License**
This project is recommended to be licensed under **Apache License 2.0**, which allows for secure open-source collaboration while protecting contributions.

---

## **✅ Summary**
This setup ensures **maximum security, private cloud storage, and seamless remote access** using **Tailscale, Caddy, and Cloudflare**.  

It is **perfect for people who cannot port forward** and allows **temporary disabling of Cloudflare proxy for large file uploads**.  

Let me know if you need **further optimizations or security improvements**! 🚀🔒
