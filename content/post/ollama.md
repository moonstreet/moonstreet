---
title: "Ollama server" # Title of the blog post.
date: 2025-02-05T16:24:21+01:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: true # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/terraform.png" # Sets featured image on blog post.
#thumbnail: "/images/path/terraform.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 50 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: true # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Technology
tags:
  - llm
  - linux
showShare: false
---

# Setting up an old school Self-Hosted Server Stack: Mattermost, OpenWebUI, and Monitoring

This is how Claude and I installed an Ollama server from scratch.

## Part 1: Installing Ollama and OpenWebUI

### Installing Ollama
First, we installed Ollama, which is the backend for our AI operations:

```bash
curl https://ollama.ai/install.sh | sh
```

After installation, we created a systemd service to manage Ollama:

```bash
sudo vim /etc/systemd/system/ollama.service
```

Added this configuration:
```ini
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=root
Restart=always

[Install]
WantedBy=multi-user.target
```

Started and enabled the service:


```bash
sudo systemctl daemon-reload
sudo systemctl enable ollama
sudo systemctl start ollama
```

To verify the installation, we pulled a model:


```bash
ollama pull phi3
```

### Setting up OpenWebUI with Ollama


OpenWebUI serves as our frontend interface for Ollama. We installed it using Python's virtual environment:

```bash
# Install required packages
sudo apt install python3-full python3-pip python3-venv

# Create and activate virtual environment
mkdir open-webui && cd open-webui
python3 -m venv openwebui-env
source openwebui-env/bin/activate

# Install OpenWebUI
pip install open-webui
```

Created a systemd service for OpenWebUI:
```bash
sudo vim /etc/systemd/system/openwebui.service
```

Added the configuration (change 'username' to your username):


```ini
[Unit]
Description=OpenWebUI
After=network.target

[Service]
Type=simple
User=username
WorkingDirectory=/home/username
Environment=PATH=/home/username/open-webui/openwebui-env/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ExecStart=/home/username/open-webui/openwebui-env/bin/open-webui serve
Restart=always

[Install]
WantedBy=multi-user.target
```

Then start it.

```sh
sudo systemctl daemon-reload
sudo systemctl enable openwebui
sudo systemctl start openwebui
```


## Part 2: Setting up Mattermost

### Initial Installation
We started by installing Mattermost on our Ubuntu server. Mattermost runs on port 8065 by default.

### Troubleshooting WebSocket Issues
Initially, we encountered the error: "Please check connection, Mattermost unreachable. If issue persists, ask administrator to check WebSocket port." This is a common issue when setting up Mattermost, especially before configuring Nginx properly.

The solution involved:
1. Checking if port 8065 was accessible:
```bash
sudo netstat -tulpn | grep 8065
```

2. Ensuring the WebSocket configuration in `config.json` matched our setup:
```json
{
    "ServiceSettings": {
        "SiteURL": "http://your-server-ip:8065",
        "WebsocketURL": "",  // Empty to use SiteURL
        "TLSStrictTransport": false
    }
}
```

3. Most importantly, properly configuring Nginx to handle WebSocket connections. The critical part in the Nginx configuration is:
```nginx
location ~ /api/v[0-9]+/(users/)?websocket$ {
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $http_host;
    # ... other proxy settings ...
}
```

### Adding SSL with Nginx
To secure our Mattermost instance, we set up Nginx as a reverse proxy and added SSL certificates:

1. Created Nginx configuration:
```bash
sudo vim /etc/nginx/sites-available/mattermost
```

Added:
```nginx
upstream backend {
    server 127.0.0.1:8065;
    keepalive 32;
}

proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=mattermost_cache:10m max_size=3g inactive=120m use_temp_path=off;

server {
    server_name mattermost.yourdomain.com;

    location ~ /api/v[0-9]+/(users/)?websocket$ {
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        client_max_body_size 50M;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Frame-Options SAMEORIGIN;
        proxy_buffers 256 16k;
        proxy_buffer_size 16k;
        proxy_read_timeout 600s;
        proxy_pass http://backend;
    }

    location / {
        client_max_body_size 50M;
        proxy_set_header Connection "";
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Frame-Options SAMEORIGIN;
        proxy_buffers 256 16k;
        proxy_buffer_size 16k;
        proxy_read_timeout 600s;
        proxy_cache mattermost_cache;
        proxy_cache_use_stale error timeout invalid_header http_500;
        proxy_cache_revalidate on;
        proxy_cache_min_uses 2;
        proxy_cache_lock on;
        proxy_cache_valid 200 7d;
        proxy_pass http://backend;
    }
}
```

### Securing Mattermost
We implemented several security measures:

1. Disabled open registration by modifying config.json:
```bash
sudo vim /opt/mattermost/config/config.json
```

Added:
```json
{
    "ServiceSettings": {
        "EnableOpenServer": false
    },
    "TeamSettings": {
        "EnableUserCreation": false,
        "EnableTeamCreation": false
    }
}
```

2. Enabled Multi-Factor Authentication:
```json
{
    "ServiceSettings": {
        "EnforceMultifactorAuthentication": true,
        "EnableMultifactorAuthentication": true
    }
}
```

3. Set up email notifications using SendGrid:
```json
{
    "EmailSettings": {
        "EnableSignUpWithEmail": true,
        "EnableSignInWithEmail": true,
        "EnableSMTPAuth": true,
        "SMTPUsername": "apikey",
        "SMTPPassword": "your-sendgrid-api-key-here",
        "SMTPServer": "smtp.sendgrid.net",
        "SMTPPort": 587,
        "ConnectionSecurity": "STARTTLS",
        "SendEmailNotifications": true,
        "FeedbackName": "Mattermost",
        "FeedbackEmail": "your-verified-sender@yourdomain.com",
        "ReplyToAddress": "your-verified-sender@yourdomain.com"
    }
}
```

## Part 3: Setting up Monitoring

### Installing Prometheus

1. Created necessary users and directories:
```bash
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

2. Downloaded and installed Prometheus:
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.49.1/prometheus-2.49.1.linux-amd64.tar.gz
tar xvf prometheus-*.tar.gz
cd prometheus-*/
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
sudo cp -r consoles/ /etc/prometheus
sudo cp -r console_libraries/ /etc/prometheus
```

3. Created Prometheus configuration:
```bash
sudo vim /etc/prometheus/prometheus.yml
```

Added basic configuration:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

### Installing Node Exporter

1. Set up Node Exporter for system metrics:
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvf node_exporter-*.tar.gz
sudo cp node_exporter-*/node_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

2. Created systemd service for Node Exporter:
```bash
sudo vim /etc/systemd/system/node_exporter.service
```

Added:
```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

### Installing Grafana

1. Added Grafana repository and installed:
```bash
sudo apt-get install -y apt-transport-https software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana
```

2. Set up Nginx as reverse proxy for Grafana:
```bash
sudo vim /etc/nginx/sites-available/grafana
```

Added:
```nginx
server {
    server_name grafana.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Useful Shell Aliases
If you're using bash/zsh, you might want to add these helpful aliases to your `.bashrc` or `.zshrc`:

```bash
# Service management
alias mm-restart='sudo systemctl restart mattermost'
alias mm-status='sudo systemctl status mattermost'
alias mm-logs='sudo journalctl -u mattermost -f'

# Quick edits
alias mm-config='sudo vim /opt/mattermost/config/config.json'
alias ng-edit='cd /etc/nginx/sites-available'

# Monitoring
alias prom-status='sudo systemctl status prometheus'
alias graf-status='sudo systemctl status grafana-server'
```

## Conclusion

We now have a fully functional, secure server stack with:
- Mattermost for team communication
- OpenWebUI for AI interactions
- Comprehensive system monitoring with Prometheus and Grafana
- Everything secured with SSL certificates
- Basic monitoring dashboards set up

Next steps could include:
- Setting up specific Mattermost monitoring
- Adding Ollama monitoring
- Creating custom Grafana dashboards
- Setting up alerting rules
- Implementing backup solutions
- Monitor token per second

Remember to regularly update all components and monitor system resources to ensure everything runs smoothly. Setting up a Self-Hosted Server Stack: Mattermost, OpenWebUI, and Monitoring

