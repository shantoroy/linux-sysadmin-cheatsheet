# Container Management 

## Docker Operations 

### Create a Docker Compose setup
```bash
# Create directory for project
mkdir -p ~/docker/webapp
cd ~/docker/webapp

# Create docker-compose.yml file
cat > docker-compose.yml << EOF
version: '3'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
      - ./nginx/conf.d:/etc/nginx/conf.d
    depends_on:
      - app
    restart: always

  app:
    image: php:7.4-fpm
    volumes:
      - ./html:/var/www/html
    environment:
      - DB_HOST=db
      - DB_NAME=webapp
      - DB_USER=webuser
      - DB_PASS=secret
    depends_on:
      - db
    restart: always

  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=rootsecret
      - MYSQL_DATABASE=webapp
      - MYSQL_USER=webuser
      - MYSQL_PASSWORD=secret
    restart: always

volumes:
  db_data:
EOF

# Create directories for volumes
mkdir -p html nginx/conf.d

# Create a custom nginx configuration
cat > nginx/conf.d/default.conf << EOF
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.php index.html;

    location / {
        try_files \$uri \$uri/ /index.php?\$args;
    }

    location ~ \.php$ {
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
        include fastcgi_params;
    }
}
EOF

# Create a test PHP file
cat > html/index.php << EOF
<?php
echo "<h1>Docker Compose Test</h1>";
echo "<p>PHP Version: " . phpversion() . "</p>";

\$host = getenv('DB_HOST');
\$dbname = getenv('DB_NAME');
\$username = getenv('DB_USER');
\$password = getenv('DB_PASS');

try {
    \$conn = new PDO("mysql:host=\$host;dbname=\$dbname", \$username, \$password);
    \$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    echo "<p>Connected to MySQL successfully!</p>";
} catch(PDOException \$e) {
    echo "<p>Connection failed: " . \$e->getMessage() . "</p>";
}
?>
EOF

# Start the Docker Compose setup
docker-compose up -d

# View logs
docker-compose logs -f

# Stop the Docker Compose setup
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

### Create a custom Docker image
```bash
# Create a directory for the Dockerfile
mkdir -p ~/docker/custom-app
cd ~/docker/custom-app

# Create a simple Python application
cat > app.py << EOF
from flask import Flask
import os

app = Flask(__name__)

@app.route('/')
def hello():
    return '<h1>Hello from Custom Docker Container!</h1>'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=int(os.environ.get('PORT', 5000)))
EOF

# Create requirements.txt
cat > requirements.txt << EOF
flask==2.0.1
EOF

# Create a Dockerfile
cat > Dockerfile << EOF
# Use official Python image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY app.py .

# Set environment variables
ENV PORT=5000
ENV PYTHONUNBUFFERED=1

# Expose the application port
EXPOSE 5000

# Start the application
CMD ["python", "app.py"]
EOF

# Build the Docker image
docker build -t custom-flask-app:latest .

# Run the container
docker run -d -p 5000:5000 --name flask-app custom-flask-app:latest

# Push to Docker Hub (if needed)
docker tag custom-flask-app:latest username/custom-flask-app:latest
docker push username/custom-flask-app:latest
```

### Copying Files to Host
1. Use docker cp (Recommended for Copying)
You can copy files or directories from the running container to your host using:

```sh
docker cp <container_id>:/app/data /path/on/host
```

Example:
```sh
docker cp my_container:/app/data ~/my_local_data
```

2. Bind Mount or Volume (Recommended for Persistent Access)
If you need continuous access, mount a host directory to /app/data when starting the container:

```sh
docker run -v /path/on/host:/app/data my_image
```


Now, any files written to /app/data inside the container will also be available in /path/on/host on your machine.

## Kubernetes Operations

### Basic kubectl commands
```bash
# View cluster information
kubectl cluster-info

# Get all resources in the cluster
kubectl get all --all-namespaces

# List nodes in the cluster
kubectl get nodes

# Describe a specific node
kubectl describe node node-name

# View all namespaces
kubectl get namespaces

# Create a new namespace
kubectl create namespace app-namespace

# Set the default namespace for kubectl
kubectl config set-context --current --namespace=app-namespace

# List all pods
kubectl get pods

# Get detailed information about a pod
kubectl describe pod pod-name

# View pod logs
kubectl logs pod-name

# View pod logs for a specific container in a pod
kubectl logs pod-name -c container-name

# Follow pod logs
kubectl logs -f pod-name

# Execute a command in a running pod
kubectl exec -it pod-name -- /bin/bash

# Get all deployments
kubectl get deployments

# Scale a deployment
kubectl scale deployment deployment-name --replicas=3

# Edit a deployment
kubectl edit deployment deployment-name

# View all services
kubectl get services

# Create a basic deployment from an image
kubectl create deployment nginx-deployment --image=nginx

# Expose a deployment as a service
kubectl expose deployment nginx-deployment --port=80 --type=LoadBalancer

# Apply a configuration from a file
kubectl apply -f deployment.yaml

# Delete a resource
kubectl delete deployment nginx-deployment
kubectl delete service nginx-deployment
```

### Create Kubernetes resources
```bash
# Create a simple deployment YAML
cat > deployment.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-app
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: "0.5"
            memory: "512Mi"
          requests:
            cpu: "0.2"
            memory: "256Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: web-app
spec:
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
EOF

# Apply the deployment
kubectl apply -f deployment.yaml

# Check the deployment
kubectl get deployments
kubectl get pods
kubectl get services

# Create a ConfigMap
cat > configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "mysql://db:3306/myapp"
  cache_host: "redis:6379"
  log_level: "info"
EOF

kubectl apply -f configmap.yaml

# Create a Secret
cat > secret.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  db_user: $(echo -n "admin" | base64)
  db_password: $(echo -n "p@ssw0rd" | base64)
  api_key: $(echo -n "a1b2c3d4e5f6g7h8i9j0" | base64)
EOF

kubectl apply -f secret.yaml

# Use ConfigMap and Secret in a Deployment
cat > app-deployment.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app:latest
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database_url
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: log_level
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db_user
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db_password
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
      volumes:
      - name: config-volume
        configMap:
          name: app-config
EOF

kubectl apply -f app-deployment.yaml
```

