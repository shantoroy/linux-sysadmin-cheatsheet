# Web Server Management

## Apache Configuration

### Set up a basic Apache virtual host
```bash
# Install Apache
apt-get install apache2

# Create directory structure
mkdir -p /var/www/example.com/public_html
chown -R www-data:www-data /var/www/example.com

# Create a basic virtual host file
cat > /etc/apache2/sites-available/example.com.conf << EOF
<VirtualHost *:80>
    ServerAdmin webmaster@example.com
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/public_html
    ErrorLog \${APACHE_LOG_DIR}/example.com_error.log
    CustomLog \${APACHE_LOG_DIR}/example.com_access.log combined
    
    <Directory /var/www/example.com/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOF

# Enable virtual host and rewrite module
a2ensite example.com.conf
a2enmod rewrite

# Test configuration and restart
apache2ctl configtest
systemctl restart apache2
```

### Configure SSL with Apache
```bash
# Enable SSL module
a2enmod ssl

# Create SSL virtual host
cat > /etc/apache2/sites-available/example.com-ssl.conf << EOF
<VirtualHost *:443>
    ServerAdmin webmaster@example.com
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/public_html
    ErrorLog \${APACHE_LOG_DIR}/example.com_error.log
    CustomLog \${APACHE_LOG_DIR}/example.com_access.log combined
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/example.com.key
    SSLCertificateChainFile /etc/ssl/certs/example.com-chain.crt
    
    <Directory /var/www/example.com/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    # HTTP Strict Transport Security
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
</VirtualHost>
EOF

# Enable headers module and the site
a2enmod headers
a2ensite example.com-ssl.conf

# Add HTTP to HTTPS redirect
cat > /etc/apache2/sites-available/example.com.conf << EOF
<VirtualHost *:80>
    ServerAdmin webmaster@example.com
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/public_html
    
    Redirect permanent / https://example.com/
</VirtualHost>
EOF

# Test configuration and restart
apache2ctl configtest
systemctl restart apache2
```

### Configure Apache for PHP
```bash
# Install PHP and necessary modules
apt-get install php php-mysql libapache2-mod-php php-cli php-common php-gd php-xmlrpc php-mbstring php-curl

# Create PHP info file for testing
echo "<?php phpinfo(); ?>" > /var/www/html/info.php

# Create a custom PHP configuration
cat > /etc/php/7.4/apache2/conf.d/custom.ini << EOF
; Basic Settings
max_execution_time = 300
memory_limit = 128M
upload_max_filesize = 20M
post_max_size = 21M
max_input_vars = 3000

; Error Handling
display_errors = Off
display_startup_errors = Off
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
log_errors = On
error_log = /var/log/php_errors.log

; Performance Settings
opcache.enable = 1
opcache.memory_consumption = 128
opcache.interned_strings_buffer = 8
opcache.max_accelerated_files = 4000
opcache.revalidate_freq = 60
EOF

# Restart Apache
systemctl restart apache2
```

### Apache performance tuning
```bash
# Install Apache utilities
apt-get install apache2-utils

# Create MPM configuration for better performance
cat > /etc/apache2/mods-available/mpm_event.conf << EOF
<IfModule mpm_event_module>
    StartServers             2
    MinSpareThreads         25
    MaxSpareThreads         75
    ThreadLimit             64
    ThreadsPerChild         25
    MaxRequestWorkers      150
    MaxConnectionsPerChild   0
</IfModule>
EOF

# Enable event MPM and disable prefork
a2dismod mpm_prefork
a2enmod mpm_event

# Enable HTTP/2
a2enmod http2
cat > /etc/apache2/conf-available/http2.conf << EOF
Protocols h2 http/1.1
EOF
a2enconf http2

# Enable cache modules
a2enmod expires
a2enmod cache
a2enmod cache_disk

# Create cache configuration
cat > /etc/apache2/conf-available/cache.conf << EOF
<IfModule mod_cache.c>
    CacheRoot /var/cache/apache2/mod_cache_disk
    CacheEnable disk /
    CacheDirLevels 2
    CacheDirLength 1
    CacheMaxExpire 86400
    CacheIgnoreNoLastMod On
    CacheIgnoreCacheControl On
    CacheDefaultExpire 3600
</IfModule>
EOF
mkdir -p /var/cache/apache2/mod_cache_disk
chown -R www-data:www-data /var/cache/apache2/mod_cache_disk
a2enconf cache

# Test configuration and restart
apache2ctl configtest
systemctl restart apache2
```

## Nginx Configuration

### Set up a basic Nginx virtual host
```bash
# Install Nginx
apt-get install nginx

# Create directory structure
mkdir -p /var/www/example.com/public_html
chown -R www-data:www-data /var/www/example.com

# Create a basic virtual host file
cat > /etc/nginx/sites-available/example.com << EOF
server {
    listen 80;
    listen [::]:80;
    
    server_name example.com www.example.com;
    root /var/www/example.com/public_html;
    index index.html index.htm index.php;
    
    location / {
        try_files \$uri \$uri/ /index.php?\$args;
    }
    
    # Pass PHP scripts to PHP-FPM
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }
    
    # Deny access to hidden files
    location ~ /\. {
        deny all;
    }
    
    # Logging
    access_log /var/log/nginx/example.com_access.log;
    error_log /var/log/nginx/example.com_error.log;
}
EOF

# Enable the site
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/

# Test configuration and restart
nginx -t
systemctl restart nginx
```

### Configure SSL with Nginx
```bash
# Create SSL virtual host
cat > /etc/nginx/sites-available/example.com << EOF
# HTTP to HTTPS redirect
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    return 301 https://\$host\$request_uri;
}

# HTTPS server block
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com www.example.com;
    
    root /var/www/example.com/public_html;
    index index.html index.htm index.php;
    
    # SSL Configuration
    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    ssl_trusted_certificate /etc/ssl/certs/example.com-chain.crt;
    
    # SSL optimizations
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;
    
    location / {
        try_files \$uri \$uri/ /index.php?\$args;
    }
    
    # Pass PHP scripts to PHP-FPM
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }
    
    # Deny access to hidden files
    location ~ /\. {
        deny all;
    }
    
    # Logging
    access_log /var/log/nginx/example.com_access.log;
    error_log /var/log/nginx/example.com_error.log;
}
EOF

# Test configuration and restart
nginx -t
systemctl restart nginx
```

### Nginx performance tuning
```bash
# Create optimized Nginx configuration
cat > /etc/nginx/nginx.conf << EOF
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 1024;
    multi_accept on;
    use epoll;
}

http {
    # Basic Settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;
    
    # Buffer size for POST submissions
    client_body_buffer_size 10K;
    client_header_buffer_size 1k;
    client_max_body_size 8m;
    large_client_header_buffers 2 1k;
    
    # Timeouts
    client_body_timeout 12;
    client_header_timeout 12;
    send_timeout 10;
    
    # File type handling
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Logging Settings
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    
    # Gzip Settings
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    
    # Security headers
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    
    # Virtual Host Configs
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
    
    # Open file cache
    open_file_cache max=2000 inactive=20s;
    open_file_cache_valid 60s;
    open_file_cache_min_uses 5;
    open_file_cache_errors off;
}
EOF

# Test configuration and restart
nginx -t
systemctl restart nginx
```

### Configure Nginx as reverse proxy
```bash
# Create a reverse proxy configuration for an application
cat > /etc/nginx/sites-available/app-proxy << EOF
server {
    listen 80;
    server_name app.example.com;
    
    # Redirect to HTTPS
    return 301 https://\$host\$request_uri;
}

server {
    listen 443 ssl http2;
    server_name app.example.com;
    
    # SSL Configuration
    ssl_certificate /etc/ssl/certs/app.example.com.crt;
    ssl_certificate_key /etc/ssl/private/app.example.com.key;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Proxy settings
    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
        proxy_cache_bypass \$http_upgrade;
    }
    
    # Logging
    access_log /var/log/nginx/app.example.com_access.log;
    error_log /var/log/nginx/app.example.com_error.log;
}
EOF

# Enable the site
ln -s /etc/nginx/sites-available/app-proxy /etc/nginx/sites-enabled/

# Test configuration and restart
nginx -t
systemctl restart nginx
```

