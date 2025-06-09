Cloudflare Tunnel Setup Guide
Cloudflare Tunnel creates a secure, outbound-only connection between your origin server (hosting your portfolio) and Cloudflare's edge network. This means you don't need to open any inbound ports on your server's firewall, and your origin IP address remains hidden.

This guide provides general steps. Always refer to the official Cloudflare Tunnel documentation for the most up-to-date and detailed instructions.

Prerequisites
A Cloudflare account.

Your domain added to Cloudflare and DNS managed by Cloudflare.

Your portfolio website running locally (e.g., via the Nginx Docker container on 127.0.0.1:8080).

Root or sudo access to your origin server.

1. Install cloudflared
cloudflared is the daemon that creates and manages the tunnel.

Follow the official installation instructions for your server's operating system (e.g., Ubuntu/Debian).

Example for Debian/Ubuntu (check for the latest package):

# Add Cloudflare's GPG key
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL [https://pkg.cloudflare.com/cloudflare-main.gpg](https://pkg.cloudflare.com/cloudflare-main.gpg) | sudo tee /usr/share/keyrings/cloudflare-main.gpg > /dev/null

# Add Cloudflare's apt repository
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] [https://pkg.cloudflare.com/cloudflared](https://pkg.cloudflare.com/cloudflared) $(lsb_release -cs) main' | sudo tee /etc/apt/sources.list.d/cloudflared.list

# Update apt and install cloudflared
sudo apt update
sudo apt install cloudflared

2. Log in cloudflared to Your Cloudflare Account
This step authenticates cloudflared and allows it to create tunnels for your account and zones.

cloudflared login

This will open a browser window asking you to log in to your Cloudflare account and select the zone (domain) you want to authorize.

3. Create a Tunnel
You can create a tunnel and give it a name.

cloudflared tunnel create your-portfolio-tunnel

This command will output:

A Tunnel ID (UUID).

A path to a credentials file (e.g., ~/.cloudflared/<Tunnel-ID>.json). Keep this file secure.

4. Configure the Tunnel to Route Traffic
You need to tell the tunnel which local service to expose and under which hostname.

Create a configuration file for cloudflared. The default location is ~/.cloudflared/config.yml, but for running as a service, it's often placed in /etc/cloudflared/config.yml.

An example config.yml might look like this:

# /etc/cloudflared/config.yml

tunnel: your-portfolio-tunnel-UUID-here # Replace with your Tunnel ID from step 3
credentials-file: /root/.cloudflared/your-portfolio-tunnel-UUID-here.json # Or /home/youruser/.cloudflared/... if not run as root

ingress:
  # Rule 1: Route traffic for portfolio.yourdomain.com to your local Nginx
  - hostname: portfolio.yourdomain.com
    service: http://localhost:8080 # Points to Nginx Docker container (port 8080 on host)

  # Rule 2: Catch-all (must be the last rule)
  # This rule ensures any other traffic to the tunnel results in an error
  - service: http_status:404

Explanation:

tunnel: The UUID of the tunnel you created.

credentials-file: Path to the JSON credentials file generated when you created the tunnel.

ingress: Defines how incoming traffic to the tunnel is routed.

hostname: The public hostname that will point to your service.

service: The local address of your web service (your Nginx container). If your docker-compose.yml maps Nginx port 80 to host port 8080 (e.g., 127.0.0.1:8080:80), then http://localhost:8080 is correct.

5. Create a DNS Record for Your Tunnel
Now, associate a public hostname (e.g., portfolio.yourdomain.com) with your tunnel using a CNAME record in Cloudflare DNS.

cloudflared tunnel route dns your-portfolio-tunnel portfolio.yourdomain.com

This command creates a CNAME record in your Cloudflare DNS settings that points portfolio.yourdomain.com to your tunnel's unique address.

6. Run the Tunnel as a Service (Recommended)
To ensure the tunnel runs persistently and starts on boot:

sudo cloudflared service install
sudo systemctl start cloudflared
sudo systemctl enable cloudflared

Check the status:

sudo systemctl status cloudflared
cloudflared tunnel list

Your tunnel should show as HEALTHY. Logs can typically be found using journalctl -u cloudflared -f.

7. Test Your Setup
Open https://portfolio.yourdomain.com in your browser. You should see your portfolio website, served through Cloudflare.

8. Security Considerations with Cloudflare Tunnel
Cloudflare WAF: Enable and configure Cloudflare's Web Application Firewall (WAF) for your domain to protect against common web threats.

Access Policies: For even tighter security, you could layer Cloudflare Access policies on top of your tunnel to restrict who can reach it.

Keep cloudflared Updated: Regularly update the cloudflared daemon on your server.

Host Firewall: Even though no inbound ports are needed for the tunnel itself, maintain your server's firewall (ufw) to restrict other unnecessary connections.

HTTPS: Cloudflare automatically provides SSL for the public-facing hostname. Your local service (http://localhost:8080) does not need to handle SSL itself, as the connection between cloudflared and your local Nginx is on the local machine.