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

**Note**: This guide assumes you're deploying on Ubuntu. Adjust commands accordingly for other distributions. Always follow security best practices in production environments. I would upload Python fastAPI Production server