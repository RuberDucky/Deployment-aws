# Ubuntu Server Deployment Guide

A comprehensive guide for deploying Node.js applications with PostgreSQL, pgAdmin, and Apache on Ubuntu servers.

## Table of Contents
- [System Updates](#system-updates)
- [Apache Installation](#apache-installation)
- [Node.js Installation](#nodejs-installation)
- [Python Installation](#python-installation)
- [PostgreSQL Setup](#postgresql-setup)
- [pgAdmin Installation](#pgadmin-installation)
- [pgVector Extension](#pgvector-extension)
- [Database Configuration](#database-configuration)
- [File Permissions](#file-permissions)
- [Systemd Service Setup](#systemd-service-setup)
- [Apache Virtual Host Configuration](#apache-virtual-host-configuration)
- [SSL Certificate Setup](#ssl-certificate-setup)
- [Troubleshooting](#troubleshooting)

## System Updates

First, update your system packages:

```bash
sudo apt update && sudo apt upgrade -y
```

## Apache Installation

Install and start Apache web server:

```bash
# Install Apache
sudo apt install apache2 -y

# Start and enable Apache
sudo systemctl start apache2
sudo systemctl enable apache2
```

## Node.js Installation

Install Node.js version 22:

```bash
# Add NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -

# Install Node.js
sudo apt install -y nodejs
```

## Python Installation

Install Python 3 and pip:

```bash
sudo apt install python3 python3-pip -y
```

## PostgreSQL Setup

Install and configure PostgreSQL:

```bash
# Install PostgreSQL
sudo apt install postgresql postgresql-contrib -y

# Start and enable PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

## pgAdmin Installation

Install pgAdmin for database management:

```bash
# Add pgAdmin repository
curl -fsS https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo gpg --dearmor -o /usr/share/keyrings/packages-pgadmin-org.gpg

# Add repository to sources
sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/packages-pgadmin-org.gpg] https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list && apt update'

# Install pgAdmin
sudo apt install pgadmin4 -y
sudo apt install pgadmin4-web -y

# Setup web interface
sudo /usr/pgadmin4/bin/setup-web.sh
```

## pgVector Extension

Install pgVector for vector operations:

```bash
# Install PostgreSQL common packages
sudo apt install -y postgresql-common

# Add PostgreSQL official repository
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

# Install pgVector extension
sudo apt install postgresql-16-pgvector
```

## Database Configuration

Configure your PostgreSQL database:

```bash
# Switch to postgres user
sudo -i -u postgres

# Access PostgreSQL
psql
```

Execute the following SQL commands:

```sql
-- Create database
CREATE DATABASE zain;

-- Set password for postgres user
ALTER USER postgres WITH PASSWORD 'root';

-- Connect to the database
\c zain;

-- Create vector extension
CREATE EXTENSION vector;

-- Exit psql
\q
```

Exit from postgres user:
```bash
exit
```

## File Permissions

Set proper permissions for web directory:

```bash
# Change ownership to ubuntu user
sudo chown -R ubuntu:ubuntu /var/www/html/

# Set permissions (Note: 777 is not recommended for production)
sudo chmod -R 777 /var/www/html/

# Navigate to web directory
cd /var/www/html
```

## Systemd Service Setup

Create a systemd service for your Node.js backend:

```bash
# Navigate to systemd directory
cd /etc/systemd/system

# Create service file
sudo nano backend.service
```

Add the following content to `backend.service`:

```ini
[Unit]
Description=NodeJs zain Backend
After=network.target

[Service]
WorkingDirectory=/var/www/html/zain/
ExecStart=/usr/bin/npm run dev
Restart=always
User=ubuntu
Environment=NODE_ENV=development
Environment=PORT=3000

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
# Reload systemd daemon
sudo systemctl daemon-reload

# Enable service to start on boot
sudo systemctl enable backend.service

# Start the service
sudo systemctl start backend.service

# Check service status
sudo systemctl status backend.service

# View service logs
sudo journalctl -u backend.service

# Follow service logs in real-time
sudo journalctl -u backend.service -f
```

## Apache Virtual Host Configuration

Navigate to Apache sites directory:

```bash
cd /etc/apache2/sites-available/
```

### Frontend + Backend Configuration

Create a virtual host file for applications with frontend:

```apache
<VirtualHost *:80>
    ServerName zaindev.robocodingit.com
    DocumentRoot /var/www/html/zain/views

    # Serve React frontend from the root
    <Directory /var/www/html/zain/views>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    # Proxy backend requests to Node.js server
    ProxyPreserveHost On
    ProxyPass /backend http://127.0.0.1:3000
    ProxyPassReverse /backend http://127.0.0.1:3000

    # Logs
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### Backend-Only Configuration

Create a virtual host file for backend-only applications:

```apache
<VirtualHost *:80>
    ServerName zaindev.example.com
    DocumentRoot /var/www/html/zain

    ProxyPreserveHost On
    ProxyPass /backend http://localhost:3000
    ProxyPassReverse /backend http://localhost:3000

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    RewriteEngine on
    RewriteCond %{SERVER_NAME} =zaindev.robocodingit.com
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

Enable Apache modules and restart:

```bash
# Enable required Apache modules
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod rewrite

# Restart Apache
sudo systemctl restart apache2
```

## SSL Certificate Setup

Install and configure SSL certificates using Certbot:

```bash
# Install Certbot
sudo apt install certbot python3-certbot-apache -y

# Obtain SSL certificate
sudo certbot --apache

# Restart services
sudo systemctl restart apache2
sudo systemctl restart backend
```

## Troubleshooting

### Common Commands

- **Check service status**: `sudo systemctl status [service-name]`
- **View logs**: `sudo journalctl -u [service-name]`
- **Restart services**: `sudo systemctl restart [service-name]`
- **Test Apache configuration**: `sudo apache2ctl configtest`

### Common Issues

1. **Port conflicts**: Ensure ports 80, 443, and 3000 are available
2. **Permission issues**: Check file ownership and permissions
3. **Database connection**: Verify PostgreSQL is running and credentials are correct
4. **SSL issues**: Ensure domain DNS is properly configured

### Useful URLs

After successful deployment, you can access:
- Your application: `https://zaindev.robocodingit.com`
- pgAdmin interface: `http://your-server-ip/pgadmin4`

---

**Note**: This guide assumes you're deploying on Ubuntu. Adjust commands accordingly for other distributions. Always follow security best practices in production environments.

---

# Python FastAPI Deployment Guide

A guide for deploying Python FastAPI applications with Nginx on Ubuntu servers.

## Table of Contents
- [Project Setup](#project-setup)
- [Virtual Environment](#virtual-environment)
- [Systemd Service for FastAPI](#systemd-service-for-fastapi)
- [Nginx Configuration](#nginx-configuration)
- [Deployment Script](#deployment-script)

## Project Setup

Ensure your FastAPI project is located in your web directory:

```bash
# Create project directory
sudo mkdir -p /var/www/insurance-housing-scheme

# Set permissions
sudo chown -R $USER:$USER /var/www/insurance-housing-scheme
```

## Virtual Environment

Create and activate a Python virtual environment:

```bash
# Navigate to project directory
cd /var/www/insurance-housing-scheme

# Create virtual environment
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Install uvicorn if not in requirements
pip install uvicorn fastapi
```

## Systemd Service for FastAPI

Create a systemd service to run your FastAPI application:

```bash
# Navigate to systemd directory
cd /etc/systemd/system

# Create service file
sudo nano insurance-housing-scheme.service
```

Add the following content to `insurance-housing-scheme.service`:

```ini
[Unit]
Description=Uvicorn instance to serve insurance-housing-scheme
After=network.target

[Service]
User=root
Group=root
WorkingDirectory=/var/www/insurance-housing-scheme
ExecStart=/var/www/insurance-housing-scheme/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Environment="PYTHONUNBUFFERED=1"
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
# Reload systemd daemon
sudo systemctl daemon-reload

# Enable service to start on boot
sudo systemctl enable insurance-housing-scheme

# Start the service
sudo systemctl start insurance-housing-scheme

# Check service status
sudo systemctl status insurance-housing-scheme

# View service logs
sudo journalctl -u insurance-housing-scheme -f
```

## Nginx Configuration

### Install Nginx

```bash
# Install Nginx
sudo apt install nginx -y

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Create Virtual Host

Create an Nginx configuration file:

```bash
sudo nano /etc/nginx/sites-available/insurance-housing-scheme
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name 52.208.233.158;  # Replace with your domain or IP

    # API requests - proxy to FastAPI
    location /api/ {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Frontend - serve static files
    location / {
        root /var/www/insurance-housing-scheme/views;
        try_files $uri $uri/ /index.html;
    }
}
```

Enable the site and restart Nginx:

```bash
# Create symbolic link to enable site
sudo ln -s /etc/nginx/sites-available/insurance-housing-scheme /etc/nginx/sites-enabled/

# Test Nginx configuration
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
```

## Deployment Script

For automated deployment, create a bash script:

```bash
sudo nano /var/www/deploy-fastapi.sh
```

Add the following content:

```bash
#!/bin/bash

SERVICE_NAME=insurance-housing-scheme
PROJECT_DIR=/var/www/insurance-housing-scheme
VENV_DIR=$PROJECT_DIR/venv
SERVICE_FILE=/etc/systemd/system/$SERVICE_NAME.service

# --- Systemd Service Setup ---
echo "Creating systemd service file at $SERVICE_FILE ..."
sudo tee $SERVICE_FILE > /dev/null <<EOF
[Unit]
Description=Uvicorn instance to serve insurance-housing-scheme
After=network.target

[Service]
User=root
Group=root
WorkingDirectory=$PROJECT_DIR
ExecStart=$VENV_DIR/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Environment="PYTHONUNBUFFERED=1"
Restart=always

[Install]
WantedBy=multi-user.target
EOF

echo "Reloading systemd daemon..."
sudo systemctl daemon-reload

echo "Enabling $SERVICE_NAME service..."
sudo systemctl enable $SERVICE_NAME

echo "Starting $SERVICE_NAME service..."
sudo systemctl start $SERVICE_NAME

echo "Service $SERVICE_NAME created, enabled, and started."

# --- Nginx Configuration ---
NGINX_CONF=/etc/nginx/sites-available/insurance-housing-scheme
NGINX_LINK=/etc/nginx/sites-enabled/insurance-housing-scheme

echo "Creating nginx config at $NGINX_CONF ..."
sudo tee $NGINX_CONF > /dev/null <<EOF
server {
    listen 80;
    server_name 52.208.233.158;

    location /api/ {
        proxy_pass http://localhost:8000;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
    }

    location / {
        root $PROJECT_DIR/views;
        try_files \$uri \$uri/ /index.html;
    }
}
EOF

echo "Creating symbolic link in sites-enabled..."
if [ ! -L "$NGINX_LINK" ]; then
    sudo ln -s $NGINX_CONF $NGINX_LINK
fi

echo "Testing nginx configuration..."
sudo nginx -t

echo "Restarting nginx..."
sudo systemctl restart nginx

echo "Nginx config created, enabled, and nginx restarted."
```

Make the script executable and run:

```bash
# Make executable
sudo chmod +x /var/www/deploy-fastapi.sh

# Run the script
sudo /var/www/deploy-fastapi.sh
```

## FastAPI Troubleshooting

### Common Commands

- **Check FastAPI service status**: `sudo systemctl status insurance-housing-scheme`
- **View FastAPI logs**: `sudo journalctl -u insurance-housing-scheme -f`
- **Restart FastAPI service**: `sudo systemctl restart insurance-housing-scheme`
- **Check Nginx status**: `sudo systemctl status nginx`
- **Test Nginx config**: `sudo nginx -t`
- **View Nginx error logs**: `sudo tail -f /var/log/nginx/error.log`

### Common Issues

1. **Port 8000 already in use**: Kill existing process with `sudo lsof -t -i:8000 | xargs kill -9`
2. **Permission denied**: Ensure proper file ownership and permissions
3. **502 Bad Gateway**: Check if FastAPI service is running
4. **Static files not loading**: Verify the `root` path in Nginx config

### SSL with Certbot (Optional)

```bash
# Install Certbot for Nginx
sudo apt install certbot python3-certbot-nginx -y

# Obtain SSL certificate
sudo certbot --nginx -d yourdomain.com

# Restart Nginx
sudo systemctl restart nginx
```

---

**Security Note**: Running services as `root` is not recommended for production. Consider creating a dedicated user for your application.