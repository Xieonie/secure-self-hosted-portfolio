🚀 Cloudflare Tunnel Setup: A Comprehensive Guide
Welcome to this guide for setting up a Cloudflare Tunnel. A Cloudflare Tunnel creates a secure, outbound-only connection between your origin server (where your website is hosted) and Cloudflare's global edge network.

The key advantage is that you don't need to open any inbound ports on your server's firewall, and your origin IP address remains hidden.

Please Note: This guide provides general steps. For the most up-to-date and detailed instructions, you should always refer to the official Cloudflare Tunnel documentation.

📋 Table of Contents
Prerequisites
Step 1: Install cloudflared
Step 2: Log in cloudflared to Your Account
Step 3: Create a Tunnel
Step 4: Configure the Tunnel
Step 5: Create a DNS Record for the Tunnel
Step 6: Run the Tunnel as a Service (Recommended)
Step 7: Test Your Setup
🛡️ Important Security Considerations
📄 Example Configuration File (config.yml)
✔️ Prerequisites
Before you begin, ensure you have the following:

Cloudflare Account: An active Cloudflare account is required.
Domain on Cloudflare: Your domain must be added to Cloudflare, with its DNS managed by Cloudflare.
Local Website: Your website must be running locally on your server (e.g., via an Nginx Docker container on 127.0.0.1:8080).
Server Access: You need root or sudo access to your origin server.
The Step-by-Step Guide
Step 1: Install cloudflared
cloudflared is the daemon that creates and manages the tunnel.

Follow the official installation instructions for your server's operating system.

Example for Debian/Ubuntu:

Bash

# Add Cloudflare's GPG key
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg > /dev/null

# Add Cloudflare's apt repository
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main' | sudo tee /etc/apt/sources.list.d/cloudflared.list

# Update apt and install cloudflared
sudo apt update
sudo apt install cloudflared
Step 2: Log in cloudflared to Your Account
This step authenticates cloudflared, allowing it to create tunnels for your account and zones (domains).

Bash

cloudflared login
A browser window will open, asking you to log in to your Cloudflare account and select the domain you wish to authorize.

Step 3: Create a Tunnel
Create a tunnel and give it a descriptive name.

Bash

cloudflared tunnel create your-portfolio-tunnel
After running this command, you will receive:

A Tunnel ID (a unique UUID).
The path to a credentials file (e.g., ~/.cloudflared/<Tunnel-ID>.json). Keep this file secure!
Step 4: Configure the Tunnel
You need to tell the tunnel which local service to expose and under which public hostname. Create a configuration file to do this.

Default location: ~/.cloudflared/config.yml
For running as a service: /etc/cloudflared/config.yml
Example content for /etc/cloudflared/config.yml:

YAML

# The UUID of the tunnel you created in step 3.
tunnel: your-portfolio-tunnel-UUID-here
# Path to the credentials file.
credentials-file: /root/.cloudflared/your-portfolio-tunnel-UUID-here.json

# Ingress rules define how incoming traffic is routed.
ingress:
  # Rule 1: Route traffic for portfolio.yourdomain.com to your local Nginx.
  - hostname: portfolio.yourdomain.com
    service: http://localhost:8080 # Points to the Nginx Docker container on host port 8080.
  
  # Rule 2: Catch-all rule (must be the last rule).
  # This ensures any other traffic results in a 404 error.
  - service: http_status:404
Configuration Explained:

tunnel: The ID of the tunnel you created.
credentials-file: The path to the .json credentials file.
ingress: Defines how incoming traffic is routed.
hostname: The public-facing hostname.
service: The local address of your web service.
Step 5: Create a DNS Record for the Tunnel
Associate your public hostname with the tunnel. This command automatically creates the correct CNAME record in your Cloudflare DNS settings.

Bash

cloudflared tunnel route dns your-portfolio-tunnel portfolio.yourdomain.com
Step 6: Run the Tunnel as a Service (Recommended)
To ensure the tunnel runs persistently and starts on boot, install it as a system service.

Bash

# Install the service
sudo cloudflared service install

# Start the service
sudo systemctl start cloudflared

# Enable the service to start on boot
sudo systemctl enable cloudflared
Check the status:

Bash

# Check the service status
sudo systemctl status cloudflared

# List all tunnels (yours should show as HEALTHY)
cloudflared tunnel list
Step 7: Test Your Setup
Open https://portfolio.yourdomain.com in your browser. You should now see your website, served securely through Cloudflare.

🛡️ Important Security Considerations
Cloudflare WAF: Enable and configure Cloudflare's Web Application Firewall (WAF) to protect your site from common web threats.
Firewall: Even though no inbound ports are needed for the tunnel, you should still maintain your server's firewall (e.g., ufw) to restrict all other unnecessary connections.
HTTPS: Cloudflare automatically provides an SSL certificate for your public hostname. Your local service (e.g., http://localhost:8080) does not need to handle SSL itself.
Keep Updated: Regularly update the cloudflared daemon on your server.
Zero Trust (Optional): For even tighter security, you can layer Cloudflare Access policies on top of your tunnel to further restrict who can reach it.
📄 Example Configuration File (config.yml)
Here is a fully commented example of a config.yml file, based on the one provided in the repository.

YAML

# Example cloudflared configuration file
# Default location: ~/.cloudflared/config.yml
# For service: /etc/cloudflared/config.yml

# The Tunnel UUID. You can obtain this by running "cloudflared tunnel list"
# or when you create the tunnel with "cloudflared tunnel create <your-tunnel-name>"
# tunnel: YOUR_TUNNEL_ID_HERE

# The path to the tunnel credentials file.
# This is typically required when running cloudflared as a service.
# Ensure the user running cloudflared has read access to this file.
# The file is usually named <YOUR_TUNNEL_ID_HERE>.json
# credentials-file: /root/.cloudflared/YOUR_TUNNEL_ID_HERE.json


# Ingress rules define how traffic is routed from the Cloudflare edge to your origin services.
ingress:
  # Rule 1: Expose your portfolio website
  # Replace 'portfolio.yourdomain.com' with your desired public hostname.
  - hostname: portfolio.yourdomain.com
    # Replace 'http://localhost:8080' with the address of your local web server.
    service: http://localhost:8080

  # Rule 2 (Optional): Expose another service on a different hostname
  # - hostname: another-service.yourdomain.com
  #   service: http://localhost:8081

  # Rule 3: Catch-all rule (must be the last rule in the ingress list)
  # This ensures that any traffic sent to the tunnel that doesn't match a hostname
  # rule above will receive a 404 error. This is good practice.
  - service: http_status:404

# Optional: Logging configuration
# loglevel: info # Can be debug, info, warn, error, fatal. Default is info.
# logfile: /var/log/cloudflared.log # Path to the log file. Default is to log to syslog/journald.