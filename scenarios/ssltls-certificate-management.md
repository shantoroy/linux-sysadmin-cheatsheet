# SSL/TLS Certificate Management

## Self-Signed Certificates

### Create a self-signed SSL certificate
```bash
# Generate private key and certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/server.key -out /etc/ssl/certs/server.crt -subj "/C=US/ST=State/L=City/O=Organization/CN=example.com"

# Set proper permissions
chmod 600 /etc/ssl/private/server.key
chmod 644 /etc/ssl/certs/server.crt
```

### Create a certificate with Subject Alternative Names (SAN)
```bash
# Create OpenSSL config
cat > openssl.cnf << EOF
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C = US
ST = State
L = City
O = Organization
CN = example.com

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
DNS.3 = mail.example.com
IP.1 = 192.168.1.10
EOF

# Generate certificate with SANs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt -config openssl.cnf -extensions v3_req

# Verify the certificate
openssl x509 -in server.crt -text -noout
```

## Lets Encrypt Certificates

### Set up certbot and obtain certificates
```bash
# Install certbot (Debian/Ubuntu)
apt-get install certbot

# For Apache
apt-get install python3-certbot-apache

# For Nginx
apt-get install python3-certbot-nginx

# Obtain certificate with Apache plugin
certbot --apache -d example.com -d www.example.com

# Obtain certificate with Nginx plugin
certbot --nginx -d example.com -d www.example.com

# Obtain certificate standalone (stop web server first)
certbot certonly --standalone -d example.com -d www.example.com

# Obtain certificate using webroot
certbot certonly --webroot -w /var/www/html -d example.com -d www.example.com

# Test auto-renewal
certbot renew --dry-run
```

## Certificate Management

### Verify and test certificates
```bash
# Check certificate information
openssl x509 -in certificate.crt -text -noout

# Verify certificate chain
openssl verify -CAfile chain.pem certificate.crt

# Check certificate expiration
openssl x509 -in certificate.crt -noout -enddate

# Check SSL/TLS of a website
openssl s_client -connect example.com:443 -servername example.com

# Quick script to check expiration of multiple domains
cat > check_ssl_expiry.sh << 'EOF'
#!/bin/bash
for domain in example.com www.example.org mail.example.net; do
  echo -n "$domain: "
  echo | openssl s_client -servername $domain -connect $domain:443 2>/dev/null | \
  openssl x509 -noout -enddate | cut -d= -f2
done
EOF
chmod +x check_ssl_expiry.sh
```

### Convert certificate formats
```bash
# Convert PEM to DER
openssl x509 -in cert.pem -outform der -out cert.der

# Convert DER to PEM
openssl x509 -in cert.der -inform der -outform pem -out cert.pem

# Convert PEM to PKCS#12 (PFX)
openssl pkcs12 -export -out certificate.pfx -inkey private.key -in certificate.crt -certfile ca-chain.crt

# Convert PKCS#12 (PFX) to PEM
openssl pkcs12 -in certificate.pfx -out certificate.pem -nodes
```

### Create a Certificate Signing Request (CSR)
```bash
# Generate private key
openssl genrsa -out private.key 2048

# Create CSR
openssl req -new -key private.key -out request.csr -subj "/C=US/ST=State/L=City/O=Organization/CN=example.com"

# Create CSR with SANs using config file
cat > csr.cnf << EOF
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C = US
ST = State
L = City
O = Organization
CN = example.com

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
DNS.3 = mail.example.com
EOF

openssl req -new -key private.key -out request.csr -config csr.cnf
```