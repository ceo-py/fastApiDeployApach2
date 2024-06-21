# Deploying FastAPI on Unix Server with Apache2

This guide outlines the steps required to deploy a FastAPI application on a Unix server using Apache2 as the reverse proxy. FastAPI is a modern, fast (high-performance), web framework for building APIs with Python 3.6+.

## Prerequisites

- Unix server (Ubuntu, CentOS, etc.)
- Apache2 installed and configured
- Python 3.6+ installed
- Let's Encrypt SSL certificate (optional but recommended)

## Step 1: Creating a Systemd Service for FastAPI

Create a systemd service to manage the FastAPI application. This allows the application to start automatically on system boot and restart in case of failure.

```ini
[Unit]
Description=Uvicorn instance to serve FastAPI app
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/path/to/application
EnvironmentFile=/path/to/venv/bin/activate
ExecStart=/path/to/venv/bin/uvicorn run_server:app --host 0.0.0.0 --port 8000 --proxy-headers
Restart=always

[Install]
WantedBy=multi-user.target
```

Make sure to replace `/path/to/application` with the directory containing your FastAPI application and adjust other paths accordingly.
To enable and start the service, run the following commands:
```bash
sudo systemctl enable your-service-name.service
sudo systemctl start your-service-name.service
```
## Step 2: Configuring Apache2

### HTTP Configuration

Create a virtual host configuration file for HTTP (port 80) to redirect traffic to HTTPS.

```apacheconf
<VirtualHost *:80>
    ServerName api.test.com

    # Redirect HTTP to HTTPS
    Redirect permanent / https://api.test.com/
</VirtualHost>
```

### HTTPS Configuration

Create a virtual host configuration file for HTTPS (port 443) to serve the FastAPI application.

```apacheconf
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName api.test.com

    SSLCertificateFile /etc/letsencrypt/live/api.test.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/api.test.com/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
</IfModule>
```

Ensure to replace `api.test.com` with your domain name and update the SSL certificate paths accordingly.
To enable the Apache virtual host configurations, run:
```bash
sudo a2ensite your-http-config.conf
sudo a2ensite your-https-config.conf
sudo systemctl reload apache2
```
## Step 3: Directory Permissions

Grant Apache2 access to the directory containing your FastAPI application.

```bash
sudo chown -R www-data:www-data /path/to/application
```

## Conclusion

You have successfully deployed your FastAPI application on a Unix server with Apache2 as the reverse proxy. Ensure to restart Apache2 for the changes to take effect.

For any issues or further customization, refer to the official documentation of FastAPI and Apache2.
