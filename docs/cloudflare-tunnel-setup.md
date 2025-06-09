Cloudflare Tunnel Setup Guide

This guide provides the complete steps to expose a local web service to the internet securely using a Cloudflare Tunnel. This method eliminates the need to open inbound firewall ports, hiding your server's IP address.

For the most current information, always consult the official Cloudflare Tunnel documentation.

Prerequisites
Before you start, you must have the following:

An active Cloudflare account.
Your domain added to your Cloudflare account.
A web application running locally on your server (e.g., on localhost:8080).
sudo or root access to your server.
Step-by-Step Configuration


Step 1: Install the cloudflared Daemon
First, install the cloudflared software that powers the tunnel on your origin server.

For Debian/Ubuntu, run the following commands:


  # Add Cloudflare's GPG key
  sudo mkdir -p --mode=0755 /usr/share/keyrings
  curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg > /dev/null

  # Add Cloudflare's apt repository
  echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main' | sudo tee /etc/apt/sources.list.d/cloudflared.list

  # Update and install
  sudo apt update
  sudo apt install cloudflared


Step 2: Authenticate with Cloudflare
This command links the cloudflared daemon to your Cloudflare account.

Bash

cloudflared login
A browser window will open. Log in and select the domain you wish to use.

Step 3: Create a Named Tunnel
Next, create the tunnel on Cloudflare's network. Give it a name you can easily remember.

Bash

cloudflared tunnel create your-portfolio-tunnel
This command will:

Output a unique Tunnel ID (UUID).
Create a credentials file (e.g., <Tunnel-ID>.json) in the ~/.cloudflared/ directory. This file is crucial and must be kept secure.
Step 4: Create the Configuration File
This file instructs cloudflared on how to route traffic from a public hostname to your local service.

Create the file at /etc/cloudflared/config.yml.
Add the following content, replacing the placeholder values:
<!-- end list -->

YAML

# /etc/cloudflared/config.yml

# The UUID of your tunnel from Step 3
tunnel: your-portfolio-tunnel-UUID-here

# The path to your credentials file
credentials-file: /root/.cloudflared/your-portfolio-tunnel-UUID-here.json

# Ingress rules specify how traffic is routed
ingress:
  # Rule 1: Point your public hostname to your local service
  - hostname: portfolio.yourdomain.com
    service: http://localhost:8080

  # Rule 2: A catch-all rule that returns a 404 for any other requests
  - service: http_status:404
Step 5: Route a Hostname to the Tunnel
Create a DNS record to point your public hostname to the newly created tunnel.

Bash

cloudflared tunnel route dns your-portfolio-tunnel portfolio.yourdomain.com
This command creates the required CNAME record automatically in your Cloudflare DNS settings.

Step 6: Run the Tunnel as a Service
Install cloudflared as a system service to ensure it runs continuously and starts automatically on boot.

Bash

# Install, start, and enable the service
sudo cloudflared service install
sudo systemctl start cloudflared
sudo systemctl enable cloudflared
You can check its status at any time:

Bash

sudo systemctl status cloudflared
Step 7: Test Your Setup
Visit https://portfolio.yourdomain.com in your web browser. Your local website should now be publicly accessible through the Cloudflare network.

Security Best Practices
Cloudflare WAF: Enable the Web Application Firewall (WAF) in your Cloudflare dashboard to protect your site from common attacks.
Server Firewall: Keep your server's firewall (like ufw) active to block all unnecessary connections, even though no inbound ports are required for the tunnel itself.
Regular Updates: Keep the cloudflared package on your server updated to the latest version.
Zero Trust Access: For sensitive applications, use Cloudflare Access policies to enforce authentication and restrict access to specific users.