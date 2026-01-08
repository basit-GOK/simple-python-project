# simple-python-project
# Flask EC2 App Deployment Guide

This guide describes how to deploy this simple Flask application to an AWS EC2 instance (Ubuntu/Amazon Linux) using Gunicorn and Nginx.

## Prerequisites

- An AWS Account
- A running EC2 instance (Ubuntu 20.04/22.04 or Amazon Linux 2 recommended)
- SSH access to your instance
- Security Group configured to allow inbound traffic on ports 22 (SSH) and 80 (HTTP)

## Setup Instructions

### 1. Update System and Install Dependencies

Connect to your EC2 instance via SSH and run:

```bash
sudo apt-get update
sudo apt-get install -y python3-pip python3-venv nginx
```

*(For Amazon Linux, use `yum` instead of `apt-get`)*

### 2. Set Up the Application

Clone your repository or copy the files to the server (e.g., in `/home/ubuntu/flask_ec2_app`).

Navigate to the project directory:

```bash
cd /home/ubuntu/flask_ec2_app
```

Create and activate a virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
```

Install Python dependencies:

```bash
pip install -r requirements.txt
```

### 3. Test Gunicorn

Test if Gunicorn can serve the app:

```bash
gunicorn --bind 0.0.0.0:5000 app:app
```

Visit `http://<your-ec2-public-ip>:5000` in your browser. You should see "Welcome to EC2".
Press `Ctrl+C` to stop Gunicorn.

### 4. Configure Systemd Service

Create a systemd service file to keep the app running:

```bash
sudo nano /etc/systemd/system/flaskapp.service
```

Add the following content (adjust paths as necessary):

```ini
[Unit]
Description=Gunicorn instance to serve flask app
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/flask_ec2_app
Environment="PATH=/home/ubuntu/flask_ec2_app/venv/bin"
ExecStart=/home/ubuntu/flask_ec2_app/venv/bin/gunicorn --workers 3 --bind unix:flaskapp.sock -m 007 app:app

[Install]
WantedBy=multi-user.target
```

Start and enable the service:

```bash
sudo systemctl start flaskapp
sudo systemctl enable flaskapp
```

Check status:
```bash
sudo systemctl status flaskapp
```

### 5. Configure Nginx

Create a new Nginx configuration block:

```bash
sudo nano /etc/nginx/sites-available/flaskapp
```

Add the following (replace `your_domain_or_ip` with your EC2 Public IP):

```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/flask_ec2_app/flaskapp.sock;
    }
}
```

Enable the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/flaskapp /etc/nginx/sites-enabled
```

Test Nginx config for errors:

```bash
sudo nginx -t
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

### 6. Final Verification

Visit `http://<your-ec2-public-ip>` in your browser. You should see "Welcome to EC2".
