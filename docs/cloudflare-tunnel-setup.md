# Cloudflare Tunnel Setup Guide

Cloudflare Tunnel creates a secure, outbound-only connection between your origin server (hosting your portfolio) and Cloudflare's edge network. This means you don't need to open any inbound ports on your server's firewall, and your origin IP address remains hidden.

This guide provides general steps. Always refer to the [official Cloudflare Tunnel documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/) for the most up-to-date and detailed instructions.

## Prerequisites

* A Cloudflare account.
* Your domain added to Cloudflare and DNS managed by Cloudflare.
* Your portfolio website running locally (e.g., via the Nginx Docker container on `127.0.0.1:8080`).
* Root or sudo access to your origin server.

## 1. Install `cloudflared`

`cloudflared` is the daemon that creates and manages the tunnel.

* Follow the [official installation instructions](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/) for your server's operating system (e.g., Ubuntu/Debian).
    Example for Debian/Ubuntu (check for the latest package):
    ```bash
    # Add Cloudflare's GPG key
    sudo mkdir -p --mode=0755 /usr/share/keyrings
    curl -fsSL [https://pkg.cloudflare.com/cloudflare-main.gpg](https://pkg.cloudflare.com/cloudflare-main.gpg) | sudo tee /usr/share/keyrings/cloudflare-main.gpg > /dev/null
    # Add Cloudflare's apt repository
    echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] [https://pkg.cloudflare.com/cloudflared](https://pkg.cloudflare.com/cloudflared) $(lsb_release -cs) main' | sudo tee /etc/apt/sources.list.d/cloudflared.list
    # Update apt and install cloudflared
    sudo apt update
    sudo apt install cloudflared
    ```

## 2. Log in `cloudflared` to Your Cloudflare Account

This step authenticates `cloudflared` and allows it to create tunnels for your account and zones.

```bash
cloudflared login

This will open a browser window asking you to log in to your Cloudflare account and select the zone (domain) you want to authorize.

3. Create a Tunnel
You can create a tunnel and give it a name.

Bash

cloudflared tunnel create your-portfolio-tunnel
This command will output:

A Tunnel ID (UUID).
A path to a credentials file (e.g., ~/.cloudflared/<Tunnel-ID>.json). Keep this file secure.
4. Configure the Tunnel to Route Traffic
You need to tell the tunnel which local service to expose and under which hostname.

Create a configuration file for cloudflared. The default location is ~/.cloudflared/config.yml, but for running as a service, it's often placed in /etc/cloudflared/config.yml.

An example config.yml (see config-examples/cloudflared-config.yml.example in this repository) might look like this:

YAML

# /etc/cloudflared/config.yml
# This config uses the tunnel created in step 3, referenced by its UUID.
# You can also specify the path to the credentials file.

tunnel: your-portfolio-tunnel-UUID-here # Replace with your Tunnel ID from step 3
credentials-file: /root/.cloudflared/your-portfolio-tunnel-UUID-here.json # Or /home/youruser/.cloudflared/... if not run as root

ingress:
  # Rule 1: Route traffic for portfolio.yourdomain.com to your local Nginx
  - hostname: portfolio.yourdomain.com
    service: http://localhost:8080 # Points to Nginx Docker container (port 8080 on host)
    # Optional: Add originRequest settings for more control if needed
    # originRequest:
    #   connectTimeout: 30s
    #   noTLSVerify: false # Set to true if your local service uses a self-signed cert (not recommended for this setup)

  # Rule 2: Catch-all (must be the last rule)
  # This rule ensures any other traffic to the tunnel results in an error
  - service: http_status:404
Explanation:

tunnel: The UUID of the tunnel you created.
credentials-file: Path to the JSON credentials file generated when you created the tunnel. Ensure this path is correct and cloudflared has permission to read it.
ingress: Defines how incoming traffic to the tunnel is routed.
hostname: The public hostname that will point to your service.
service: The local address of your web service (your Nginx container). If your docker-compose.yml maps Nginx port 80 to host port 8080 (e.g., 127.0.0.1:8080:80), then http://localhost:8080 is correct.
5. Create a DNS Record for Your Tunnel
Now, associate a public hostname (e.g., portfolio.yourdomain.com) with your tunnel using a CNAME record in Cloudflare DNS.

Bash

cloudflared tunnel route dns your-portfolio-tunnel portfolio.yourdomain.com
# Or, if you used a different name than the tunnel ID for the tunnel itself:
# cloudflared tunnel route dns <TUNNEL_NAME_OR_UUID> <public_hostname>
This command creates a CNAME record in your Cloudflare DNS settings that points portfolio.yourdomain.com to your tunnel's unique address.

6. Run the Tunnel as a Service (Recommended)
To ensure the tunnel runs persistently and starts on boot:

Bash

sudo cloudflared service install
sudo systemctl start cloudflared
sudo systemctl enable cloudflared
Check the status:

Bash

sudo systemctl status cloudflared
cloudflared tunnel list # Should show your tunnel as HEALTHY
Logs can typically be found using journalctl -u cloudflared -f.

7. Test Your Setup
Open https://portfolio.yourdomain.com in your browser. You should see your portfolio website, served through Cloudflare.

Security Considerations with Cloudflare Tunnel
Cloudflare WAF: Enable and configure Cloudflare's Web Application Firewall (WAF) for your domain to protect against common web threats.
Access Policies (Cloudflare Zero Trust): For even tighter security (though perhaps overkill for a public portfolio), you could layer Cloudflare Access policies on top of your tunnel to restrict who can reach it.
Keep cloudflared Updated: Regularly update the cloudflared daemon on your server.
Firewall: Even though no inbound ports are needed for the tunnel itself, maintain your server's firewall (ufw) to restrict other unnecessary outbound and inbound connections.
HTTPS: Cloudflare automatically provides SSL for the public-facing hostname. Your local service (http://localhost:8080) does not need to handle SSL itself, as the connection between cloudflared and your local Nginx is on the local machine or a trusted Docker network.
This setup provides a secure way to host your portfolio without exposing your server directly to the internet.


---
**File: `config-examples/cloudflared-config.yml.example`**
---
```yaml
# Example cloudflared configuration file
# Default location: ~/.cloudflared/config.yml
# For service: /etc/cloudflared/config.yml

# The Tunnel UUID. You can obtain this by running "cloudflared tunnel list"
# or when you create the tunnel with "cloudflared tunnel create <your-tunnel-name>"
# tunnel: YOUR_TUNNEL_ID_HERE

# Alternatively, if you named your tunnel and want to run it by name using the
# credentials file from the default location (~/.cloudflared/<tunnel-name>.json or /root/.cloudflared/<tunnel-name>.json):
# tunnel: your-tunnel-name

# The path to the tunnel credentials file.
# This is typically required when running cloudflared as a service,
# especially if the service user is different from the user who created the tunnel.
# The file is usually named <YOUR_TUNNEL_ID_HERE>.json
# Ensure the user running cloudflared has read access to this file.
# credentials-file: /root/.cloudflared/YOUR_TUNNEL_ID_HERE.json
# Or for a non-root user:
# credentials-file: /home/youruser/.cloudflared/YOUR_TUNNEL_ID_HERE.json


# Ingress rules define how traffic is routed from the Cloudflare edge to your origin services.
# Each rule maps a public hostname to a local service.
ingress:
  # Rule 1: Expose your portfolio website
  # Replace 'portfolio.yourdomain.com' with your desired public hostname.
  - hostname: portfolio.yourdomain.com
    # Replace 'http://localhost:8080' with the address of your local web server.
    # If your Nginx Docker container is mapped to host port 8080 (e.g., 127.0.0.1:8080:80),
    # then http://localhost:8080 or http://127.0.0.1:8080 is correct.
    service: http://localhost:8080
    # Optional: Origin specific settings
    # originRequest:
      # How long cloudflared should wait for a TCP connection to your origin server.
      # connectTimeout: 30s
      # How long cloudflared should wait for headers from your origin server.
      # http خانوادهHeadersTimeout: 60s
      # Disables TLS verification for connections to your origin server.
      # Only use this if your origin server uses a self-signed certificate and you understand the risks.
      # For this Nginx setup, Nginx serves HTTP locally, so this is not needed.
      # noTLSVerify: false

  # Rule 2 (Optional): Expose another service on a different hostname
  # - hostname: another-service.yourdomain.com
  #   service: http://localhost:8081

  # Rule 3: Catch-all rule (must be the last rule in the ingress list)
  # This ensures that any traffic sent to the tunnel that doesn't match a hostname
  # rule above will receive a 404 error. This is good practice.
  - service: http_status:404

# Optional: Warp routing configuration (for connecting private networks)
# Not typically used for exposing a public website.
# warp-routing:
#  enabled: false

# Optional: Logging configuration
# loglevel: info # Can be debug, info, warn, error, fatal. Default is info.
# logfile: /var/log/cloudflared.log # Path to the log file. Default is to log to syslog/journald.
# transport-loglevel: info # Loglevel for the transport layer.

# Optional: Autoupdate frequency (e.g., 24h)
# autoupdate-freq: 24h