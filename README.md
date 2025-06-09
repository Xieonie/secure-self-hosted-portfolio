# Secure Self-Hosted Portfolio Website 🖼️🔒

This repository documents the development, hosting, and security hardening of my portfolio website. The website is served from a hardened Ubuntu Server using Nginx as the webserver. It's securely exposed to the internet via a Cloudflare Tunnel, with the Nginx instance running inside a Docker container for encrypted connections and additional security measures like Cloudflare's WAF.

## 🎯 Goals

* Deploy a performant and highly available portfolio website.
* Maximize security through server hardening, encrypted connections, and a Web Application Firewall (WAF).
* Obscure the origin server's IP address using Cloudflare Tunnel.
* Simplify website maintenance and updates through Docker containerization.
* Implement web security best practices.

## 🛠️ Technologies Used

* [Ubuntu Server](https://ubuntu.com/server) (hardened)
* [Nginx](https://www.nginx.com/) (as web server)
* [Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/)
* [Cloudflare Tunnel (Argo Tunnel)](https://www.cloudflare.com/products/tunnel/)
* [Cloudflare WAF](https://www.cloudflare.com/waf/) (Web Application Firewall)
* SSL/TLS certificates (managed by Cloudflare)
* HTML, CSS, JavaScript (for the website itself)

## ✨ Key Features/Highlights

* **Server Hardening:** Ubuntu Server configured according to security best practices (e.g., `ufw` firewall, `fail2ban`, regular updates, minimized software installation). See [`docs/server-hardening-checklist.md`](./docs/server-hardening-checklist.md).
* **Nginx Web Server:** Efficiently serves static website content.
* **Docker Containerization:** Nginx runs within a Docker container for better isolation, portability, and ease of management.
* **Cloudflare Tunnel:** Establishes a secure, outbound-only connection from the server to Cloudflare, eliminating the need to open inbound ports on the firewall. The origin server's IP remains hidden.
* **SSL/TLS Encryption:** End-to-end encryption of traffic, managed by Cloudflare.
* **Web Application Firewall (WAF):** Utilizes Cloudflare's WAF to protect against common web attacks (e.g., SQL injection, XSS).
* **Hidden Origin IP:** Enhanced security and DDoS protection provided by Cloudflare.
* **Simple Deployment:** Website changes can be deployed by updating the static files in the `src/` directory and, if necessary, rebuilding/restarting the Docker container.

## 🏛️ Repository Structure
secure-self-hosted-portfolio/
├── README.md
├── docker-compose.yml
├── nginx/
│   ├── nginx.conf
│   └── Dockerfile
├── src/
│   └── html/         # Your website's static files go here
│       ├── index.html
│       ├── css/
│       │   └── style.css
│       └── js/
│           └── script.js
├── docs/
│   ├── server-hardening-checklist.md
│   └── cloudflare-tunnel-setup.md
└── config-examples/
└── cloudflared-config.yml.example


## 🚀 Getting Started / Configuration

This repository contains configuration examples and guides. The actual source code for your portfolio website should be placed in the `src/html/` directory.

1.  **Server Setup:**
    * Install and harden an Ubuntu Server. Refer to the [`docs/server-hardening-checklist.md`](./docs/server-hardening-checklist.md).
    * Install Docker and Docker Compose.
2.  **Cloudflare Setup:**
    * Create a Cloudflare account and add your domain.
    * Set up a Cloudflare Tunnel. Follow the [official Cloudflare documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/).
    * Configure DNS records and WAF settings in your Cloudflare dashboard. See [`docs/cloudflare-tunnel-setup.md`](./docs/cloudflare-tunnel-setup.md) for guidance.
3.  **Nginx & Docker:**
    * The `docker-compose.yml`, `nginx/nginx.conf`, and `nginx/Dockerfile` in this repository serve as templates.
    * Place your static website files (HTML, CSS, JavaScript, images, etc.) into the `src/html/` directory.
    * Build and start the Nginx container:
        ```bash
        docker-compose up --build -d
        ```
4.  **Connect Cloudflare Tunnel to Nginx:**
    * Install the `cloudflared` daemon on your Ubuntu server (or run it as another Docker container).
    * Configure `cloudflared` to point to your Nginx service, which is exposed by Docker Compose (e.g., `http://127.0.0.1:8080` if you map port 80 of the Nginx container to 8080 on the host).
    * An example `cloudflared` configuration can be found in [`config-examples/cloudflared-config.yml.example`](./config-examples/cloudflared-config.yml.example).

## 📄 Important Files

* `docker-compose.yml`: Defines the Nginx service.
* `nginx/nginx.conf`: Nginx configuration file.
* `nginx/Dockerfile`: Dockerfile to build a custom Nginx image (e.g., to copy in the `nginx.conf`).
* `src/html/`: Location for your portfolio's static website files.
* `docs/`: Contains setup and hardening guides.
* `config-examples/`: Example configurations.

## 🔮 Potential Improvements/Future Plans

* Implement Content Security Policy (CSP) headers for enhanced security.
* Set up a CI/CD pipeline for automatic deployment on Git push.
* Conduct regular security audits and (simulated) penetration tests.
* Optimize website performance (e.g., image compression, advanced caching strategies).
* Add more detailed monitoring for the Nginx container and website availability.