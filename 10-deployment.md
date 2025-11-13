# Deployment Guide: From Zero to Hero

A comprehensive guide for deploying Flutter applications, covering manual deployment, Docker containerization, and Kubernetes orchestration.

## Table of Contents
1. [Manual Deployment from Scratch](#1-manual-deployment-from-scratch)
2. [Docker: Zero to Hero](#2-docker-zero-to-hero)
3. [Kubernetes: Container Orchestration](#3-kubernetes-container-orchestration)
4. [CI/CD Integration](#4-cicd-integration)
5. [Production Best Practices](#5-production-best-practices)

---

## 1. Manual Deployment from Scratch

### 1.1 Understanding Flutter Build Artifacts

Flutter supports multiple platforms. Each platform has different deployment requirements:

- **Android**: APK/AAB files
- **iOS**: IPA files
- **Web**: HTML/JS/CSS static files
- **Desktop**: Platform-specific executables

### 1.2 Manual Android Deployment

#### Step 1: Configure Your App

**pubspec.yaml**
```yaml
name: my_app
version: 1.0.0+1  # version+buildNumber
description: My Flutter application

environment:
  sdk: '>=3.0.0 <4.0.0'
```

#### Step 2: Configure App Signing

Create `android/key.properties`:
```properties
storePassword=your_keystore_password
keyPassword=your_key_password
keyAlias=your_key_alias
storeFile=/path/to/your/keystore.jks
```

**android/app/build.gradle**
```gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    compileSdkVersion 34

    defaultConfig {
        applicationId "com.example.myapp"
        minSdkVersion 21
        targetSdkVersion 34
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName
    }

    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

#### Step 3: Generate Keystore (First Time Only)

```bash
# Generate a keystore
keytool -genkey -v -keystore ~/my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-key-alias

# Verify keystore
keytool -list -v -keystore ~/my-release-key.jks
```

#### Step 4: Build Release APK/AAB

```bash
# Build APK (for direct installation)
flutter build apk --release

# Build App Bundle (for Google Play Store)
flutter build appbundle --release

# Build with specific flavor
flutter build apk --release --flavor production

# Build with split per ABI (smaller APKs)
flutter build apk --release --split-per-abi
```

Build artifacts location:
- APK: `build/app/outputs/flutter-apk/app-release.apk`
- AAB: `build/app/outputs/bundle/release/app-release.aab`

#### Step 5: Manual Installation & Distribution

```bash
# Install on connected device
adb install build/app/outputs/flutter-apk/app-release.apk

# Install on specific device
adb -s device_id install build/app/outputs/flutter-apk/app-release.apk

# Upload to Google Play Console (manual)
# 1. Go to play.google.com/console
# 2. Create app or select existing
# 3. Navigate to Production > Create new release
# 4. Upload app-release.aab
# 5. Fill release notes
# 6. Submit for review
```

### 1.3 Manual iOS Deployment

#### Step 1: Configure Xcode Project

```bash
# Open iOS project in Xcode
open ios/Runner.xcworkspace
```

In Xcode:
1. Select Runner project
2. Update Bundle Identifier
3. Select Team (requires Apple Developer account)
4. Configure signing & capabilities

#### Step 2: Build iOS App

```bash
# Build iOS app
flutter build ios --release

# Build for specific configuration
flutter build ios --release --flavor production
```

#### Step 3: Create Archive in Xcode

1. In Xcode: Product > Archive
2. Once archived, click "Distribute App"
3. Choose distribution method:
   - **App Store Connect**: For App Store
   - **Ad Hoc**: For testing on registered devices
   - **Enterprise**: For internal distribution
   - **Development**: For development testing

#### Step 4: Upload to App Store Connect

```bash
# Using Xcode (GUI)
# 1. Window > Organizer
# 2. Select your archive
# 3. Click "Distribute App"
# 4. Follow wizard to upload to App Store Connect

# Using command line (requires fastlane)
fastlane deliver
```

### 1.4 Manual Web Deployment

#### Step 1: Build Web App

```bash
# Build for web
flutter build web --release

# Build with custom base href
flutter build web --base-href /my-app/

# Build with web renderer (choose based on your needs)
flutter build web --web-renderer canvaskit  # Better performance
flutter build web --web-renderer html       # Smaller download size
```

Build output: `build/web/`

#### Step 2: Deploy to Web Server

**Example 1: Deploy to Apache/Nginx**

```bash
# Copy files to web server
scp -r build/web/* user@server:/var/www/html/my-app/

# Or using rsync
rsync -avz build/web/ user@server:/var/www/html/my-app/
```

**Nginx Configuration:**
```nginx
server {
    listen 80;
    server_name myapp.example.com;
    root /var/www/html/my-app;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Enable gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```

**Example 2: Deploy to Firebase Hosting**

```bash
# Install Firebase CLI
npm install -g firebase-tools

# Login to Firebase
firebase login

# Initialize Firebase in your project
firebase init hosting

# Deploy
firebase deploy --only hosting
```

**Example 3: Deploy to Netlify**

```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login to Netlify
netlify login

# Deploy
netlify deploy --dir=build/web --prod
```

**Example 4: Deploy to GitHub Pages**

```bash
# Build with base href
flutter build web --base-href /repository-name/

# Deploy using gh-pages (requires package)
npm install -g gh-pages
gh-pages -d build/web
```

### 1.5 Manual Desktop Deployment

#### Linux

```bash
# Build Linux app
flutter build linux --release

# Output: build/linux/x64/release/bundle/

# Create installer (optional, requires snapcraft)
snapcraft
```

#### macOS

```bash
# Build macOS app
flutter build macos --release

# Output: build/macos/Build/Products/Release/my_app.app

# Create DMG (requires create-dmg tool)
create-dmg build/macos/Build/Products/Release/my_app.app
```

#### Windows

```bash
# Build Windows app
flutter build windows --release

# Output: build/windows/runner/Release/

# Create installer using Inno Setup or NSIS
```

---

## 2. Docker: Zero to Hero

### 2.1 What is Docker?

**Docker** is a platform that allows you to package applications and their dependencies into containers. Think of containers as lightweight, portable, self-sufficient units that can run anywhere.

**Key Concepts:**
- **Image**: A blueprint for containers (like a class in OOP)
- **Container**: A running instance of an image (like an object)
- **Dockerfile**: Instructions to build an image
- **Registry**: Storage for images (e.g., Docker Hub)
- **Volume**: Persistent data storage
- **Network**: Container communication

**Android/Kotlin Analogy:**
```
Docker Image    →   APK file
Docker Container →  Running app instance
Dockerfile      →   build.gradle
Docker Hub      →   Maven Central/Google's Maven Repository
```

### 2.2 Installing Docker

**Linux:**
```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Verify installation
docker --version
docker run hello-world
```

**macOS:**
```bash
# Using Homebrew
brew install --cask docker

# Or download Docker Desktop from docker.com
```

**Windows:**
```bash
# Download and install Docker Desktop from docker.com
# Requires WSL2 for Windows 10/11
```

### 2.3 Docker Basics - Essential Commands

```bash
# ===== IMAGE COMMANDS =====

# Pull an image from registry
docker pull nginx:latest

# List local images
docker images

# Build an image from Dockerfile
docker build -t my-app:1.0 .

# Remove an image
docker rmi image-name

# Tag an image
docker tag my-app:1.0 myregistry/my-app:1.0

# Push image to registry
docker push myregistry/my-app:1.0


# ===== CONTAINER COMMANDS =====

# Run a container
docker run nginx

# Run container in detached mode (background)
docker run -d nginx

# Run with port mapping (host:container)
docker run -d -p 8080:80 nginx

# Run with name
docker run -d --name my-nginx nginx

# Run with environment variables
docker run -d -e MY_VAR=value nginx

# Run with volume mount
docker run -d -v /host/path:/container/path nginx

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container
docker stop container-id

# Start a stopped container
docker start container-id

# Restart a container
docker restart container-id

# Remove a container
docker rm container-id

# Remove all stopped containers
docker container prune

# View container logs
docker logs container-id

# Follow logs in real-time
docker logs -f container-id

# Execute command in running container
docker exec -it container-id /bin/bash

# View container resource usage
docker stats

# Inspect container details
docker inspect container-id


# ===== SYSTEM COMMANDS =====

# View Docker disk usage
docker system df

# Clean up unused resources
docker system prune

# Clean up everything (careful!)
docker system prune -a --volumes
```

### 2.4 Creating Your First Dockerfile

**Simple Example - Flutter Web App:**

```dockerfile
# Dockerfile for Flutter Web

# Stage 1: Build the Flutter app
FROM ubuntu:22.04 AS build

# Install dependencies
RUN apt-get update && apt-get install -y \
    curl \
    git \
    unzip \
    xz-utils \
    zip \
    libglu1-mesa \
    && rm -rf /var/lib/apt/lists/*

# Install Flutter
RUN git clone https://github.com/flutter/flutter.git /flutter
ENV PATH="/flutter/bin:${PATH}"

# Enable Flutter web
RUN flutter config --enable-web
RUN flutter doctor

# Set working directory
WORKDIR /app

# Copy project files
COPY pubspec.* ./
RUN flutter pub get

COPY . .

# Build web app
RUN flutter build web --release

# Stage 2: Serve with Nginx
FROM nginx:alpine

# Copy built files from build stage
COPY --from=build /app/build/web /usr/share/nginx/html

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 80
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

**nginx.conf:**
```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```

### 2.5 Building and Running Your Dockerized App

```bash
# Build the image
docker build -t my-flutter-app:1.0 .

# Run the container
docker run -d -p 8080:80 --name flutter-app my-flutter-app:1.0

# Test the app
curl http://localhost:8080

# View logs
docker logs -f flutter-app

# Stop and remove
docker stop flutter-app
docker rm flutter-app
```

### 2.6 Advanced Dockerfile - Multi-stage Build

**Complete Example with Backend API:**

```dockerfile
# Dockerfile for Flutter Web + Backend API

# ===== STAGE 1: Build Flutter Web App =====
FROM ubuntu:22.04 AS flutter-build

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    curl git unzip xz-utils zip libglu1-mesa \
    && rm -rf /var/lib/apt/lists/*

RUN git clone https://github.com/flutter/flutter.git -b stable /flutter
ENV PATH="/flutter/bin:/flutter/bin/cache/dart-sdk/bin:${PATH}"

RUN flutter config --enable-web
RUN flutter doctor -v

WORKDIR /app
COPY pubspec.* ./
RUN flutter pub get

COPY . .
RUN flutter build web --release --web-renderer canvaskit


# ===== STAGE 2: Build Dart Backend =====
FROM dart:stable AS dart-build

WORKDIR /backend
COPY backend/ ./

RUN dart pub get
RUN dart compile exe bin/server.dart -o bin/server


# ===== STAGE 3: Final Production Image =====
FROM nginx:alpine

# Install runtime dependencies
RUN apk add --no-cache bash

# Copy Flutter web app
COPY --from=flutter-build /app/build/web /usr/share/nginx/html

# Copy Dart backend binary
COPY --from=dart-build /backend/bin/server /app/server

# Copy nginx configuration
COPY docker/nginx.conf /etc/nginx/conf.d/default.conf

# Copy startup script
COPY docker/start.sh /start.sh
RUN chmod +x /start.sh

EXPOSE 80 8080

CMD ["/start.sh"]
```

**start.sh:**
```bash
#!/bin/bash

# Start backend API
/app/server &

# Start nginx
nginx -g "daemon off;"
```

### 2.7 Docker Compose - Managing Multiple Containers

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Flutter Web App
  frontend:
    build:
      context: .
      dockerfile: Dockerfile.frontend
    ports:
      - "3000:80"
    environment:
      - API_URL=http://backend:8080
    depends_on:
      - backend
    networks:
      - app-network

  # Backend API
  backend:
    build:
      context: .
      dockerfile: Dockerfile.backend
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
      - JWT_SECRET=your-secret-key
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs
    networks:
      - app-network

  # PostgreSQL Database
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

  # Redis Cache
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - app-network

  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - backend
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

**Using Docker Compose:**

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f backend

# Stop all services
docker-compose down

# Stop and remove volumes (deletes data!)
docker-compose down -v

# Rebuild and restart
docker-compose up -d --build

# Scale a service
docker-compose up -d --scale backend=3

# Execute command in service
docker-compose exec backend /bin/bash
```

### 2.8 Docker Best Practices

**1. Use Multi-stage Builds**
```dockerfile
# Good: Reduces final image size
FROM dart:stable AS build
WORKDIR /app
COPY . .
RUN dart compile exe bin/server.dart -o server

FROM scratch
COPY --from=build /app/server /server
CMD ["/server"]
```

**2. Optimize Layer Caching**
```dockerfile
# Good: Copy dependency files first
COPY pubspec.* ./
RUN flutter pub get

# Then copy source code
COPY . .
RUN flutter build web
```

**3. Use .dockerignore**
```
# .dockerignore
.git
.github
build/
.dart_tool/
.idea/
*.md
test/
*.log
```

**4. Don't Run as Root**
```dockerfile
# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
```

**5. Use Specific Image Versions**
```dockerfile
# Bad
FROM ubuntu:latest

# Good
FROM ubuntu:22.04
```

**6. Minimize Layer Count**
```dockerfile
# Good: Chain commands
RUN apt-get update && \
    apt-get install -y curl git && \
    rm -rf /var/lib/apt/lists/*
```

**7. Health Checks**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

### 2.9 Docker Registry & Distribution

**Push to Docker Hub:**
```bash
# Login
docker login

# Tag image
docker tag my-app:1.0 yourusername/my-app:1.0

# Push
docker push yourusername/my-app:1.0

# Pull on another machine
docker pull yourusername/my-app:1.0
```

**Private Registry:**
```bash
# Run private registry
docker run -d -p 5000:5000 --name registry registry:2

# Tag for private registry
docker tag my-app:1.0 localhost:5000/my-app:1.0

# Push to private registry
docker push localhost:5000/my-app:1.0
```

---

## 3. Kubernetes: Container Orchestration

### 3.1 What is Kubernetes?

**Kubernetes (K8s)** is a container orchestration platform that automates deployment, scaling, and management of containerized applications.

**Why Kubernetes?**
- **Automatic scaling**: Scale up/down based on load
- **Self-healing**: Automatically replaces failed containers
- **Load balancing**: Distributes traffic across containers
- **Rolling updates**: Zero-downtime deployments
- **Service discovery**: Automatic networking between services
- **Secret management**: Secure configuration management

**Docker vs Kubernetes:**
```
Docker           →  Runs containers on a single machine
Docker Compose   →  Manages multiple containers on a single machine
Kubernetes       →  Manages containers across multiple machines (cluster)
```

### 3.2 Kubernetes Architecture

```
┌─────────────────────────────────────────────────────┐
│              Kubernetes Cluster                      │
│                                                      │
│  ┌──────────────────────────────────────────────┐  │
│  │         Control Plane (Master)                │  │
│  │  - API Server                                 │  │
│  │  - Scheduler                                  │  │
│  │  - Controller Manager                         │  │
│  │  - etcd (database)                            │  │
│  └──────────────────────────────────────────────┘  │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐ │
│  │   Node 1     │  │   Node 2     │  │  Node 3   │ │
│  │              │  │              │  │           │ │
│  │ - Pod 1      │  │ - Pod 3      │  │ - Pod 5   │ │
│  │ - Pod 2      │  │ - Pod 4      │  │ - Pod 6   │ │
│  │              │  │              │  │           │ │
│  │ - kubelet    │  │ - kubelet    │  │ - kubelet │ │
│  │ - kube-proxy │  │ - kube-proxy │  │- kube-proxy│ │
│  └──────────────┘  └──────────────┘  └───────────┘ │
└─────────────────────────────────────────────────────┘
```

**Key Components:**

1. **Pod**: Smallest deployable unit (1+ containers)
2. **Node**: A machine in the cluster (physical or virtual)
3. **Service**: Stable endpoint to access Pods
4. **Deployment**: Manages Pod lifecycle and scaling
5. **ConfigMap**: Configuration data
6. **Secret**: Sensitive data (passwords, keys)
7. **Ingress**: HTTP/HTTPS routing to services
8. **Namespace**: Virtual clusters within a cluster

### 3.3 Installing Kubernetes

**Option 1: Minikube (Local Development)**

```bash
# Install Minikube (Linux)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Start Minikube
minikube start

# Verify
kubectl get nodes
kubectl cluster-info
```

**Option 2: Kind (Kubernetes in Docker)**

```bash
# Install Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create cluster
kind create cluster --name my-cluster

# Verify
kubectl get nodes
```

**Option 3: Cloud Providers**

```bash
# Google Kubernetes Engine (GKE)
gcloud container clusters create my-cluster --num-nodes=3

# Amazon EKS
eksctl create cluster --name my-cluster --region us-west-2

# Azure AKS
az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 3
```

### 3.4 Kubernetes Basic Commands

```bash
# ===== CLUSTER INFO =====
kubectl cluster-info
kubectl get nodes
kubectl describe node node-name


# ===== PODS =====
kubectl get pods
kubectl get pods -o wide
kubectl get pods -n namespace-name
kubectl describe pod pod-name
kubectl logs pod-name
kubectl logs -f pod-name  # Follow logs
kubectl exec -it pod-name -- /bin/bash
kubectl delete pod pod-name


# ===== DEPLOYMENTS =====
kubectl get deployments
kubectl describe deployment deployment-name
kubectl create deployment my-app --image=my-app:1.0
kubectl scale deployment my-app --replicas=5
kubectl delete deployment deployment-name


# ===== SERVICES =====
kubectl get services
kubectl describe service service-name
kubectl expose deployment my-app --port=80 --type=LoadBalancer
kubectl delete service service-name


# ===== APPLY YAML CONFIGS =====
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f .  # Apply all YAML files in directory


# ===== NAMESPACES =====
kubectl get namespaces
kubectl create namespace dev
kubectl get pods -n dev
kubectl delete namespace dev


# ===== CONFIGMAPS & SECRETS =====
kubectl create configmap my-config --from-literal=key=value
kubectl create secret generic my-secret --from-literal=password=secret123
kubectl get configmaps
kubectl get secrets
kubectl describe configmap my-config


# ===== DEBUGGING =====
kubectl get events
kubectl top nodes
kubectl top pods
kubectl port-forward pod-name 8080:80
```

### 3.5 Deploying Flutter App to Kubernetes

**Step 1: Create Docker Image**

```bash
# Build and push to registry
docker build -t yourusername/flutter-app:1.0 .
docker push yourusername/flutter-app:1.0
```

**Step 2: Create Deployment**

**deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flutter-app
  labels:
    app: flutter-app
spec:
  replicas: 3  # Run 3 instances
  selector:
    matchLabels:
      app: flutter-app
  template:
    metadata:
      labels:
        app: flutter-app
    spec:
      containers:
      - name: flutter-app
        image: yourusername/flutter-app:1.0
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        env:
        - name: API_URL
          value: "https://api.example.com"
        - name: ENVIRONMENT
          value: "production"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

**Step 3: Create Service**

**service.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: flutter-app-service
spec:
  type: LoadBalancer  # Use NodePort for Minikube
  selector:
    app: flutter-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    # nodePort: 30080  # Uncomment for NodePort type
```

**Step 4: Deploy to Kubernetes**

```bash
# Apply configurations
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Check status
kubectl get deployments
kubectl get pods
kubectl get services

# Get service URL (Minikube)
minikube service flutter-app-service --url

# Get service URL (Cloud)
kubectl get service flutter-app-service -o wide
```

### 3.6 Complete Production Setup

**1. ConfigMap for Configuration**

**configmap.yaml:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: flutter-app-config
data:
  API_URL: "https://api.production.com"
  TIMEOUT: "30"
  MAX_RETRIES: "3"
  nginx.conf: |
    server {
        listen 80;
        root /usr/share/nginx/html;
        index index.html;

        location / {
            try_files $uri $uri/ /index.html;
        }

        location /api {
            proxy_pass http://backend-service:8080;
        }
    }
```

**2. Secret for Sensitive Data**

```bash
# Create secret from literal
kubectl create secret generic flutter-app-secret \
  --from-literal=api-key=your-api-key \
  --from-literal=db-password=your-password

# Or from file
kubectl create secret generic flutter-app-secret \
  --from-file=./secrets/api-key.txt
```

**secret.yaml:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: flutter-app-secret
type: Opaque
data:
  api-key: eW91ci1hcGkta2V5LWJhc2U2NA==  # base64 encoded
  db-password: eW91ci1wYXNzd29yZA==
```

**3. Updated Deployment with ConfigMap and Secret**

**deployment-full.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flutter-app
  labels:
    app: flutter-app
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: flutter-app
  template:
    metadata:
      labels:
        app: flutter-app
        version: v1
    spec:
      containers:
      - name: flutter-app
        image: yourusername/flutter-app:1.0
        imagePullPolicy: Always
        ports:
        - containerPort: 80
          name: http

        # Environment variables from ConfigMap
        envFrom:
        - configMapRef:
            name: flutter-app-config

        # Environment variables from Secret
        env:
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: flutter-app-secret
              key: api-key
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: flutter-app-secret
              key: db-password

        # Volume mounts
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/conf.d
          readOnly: true
        - name: cache
          mountPath: /var/cache/nginx

        # Resource limits
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"

        # Health checks
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3

        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3

      # Volumes
      volumes:
      - name: nginx-config
        configMap:
          name: flutter-app-config
          items:
          - key: nginx.conf
            path: default.conf
      - name: cache
        emptyDir: {}

      # Node selector (optional)
      nodeSelector:
        disktype: ssd

      # Tolerations (optional)
      tolerations:
      - key: "key1"
        operator: "Equal"
        value: "value1"
        effect: "NoSchedule"
```

**4. Horizontal Pod Autoscaler**

**hpa.yaml:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: flutter-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: flutter-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**5. Ingress for External Access**

**ingress.yaml:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: flutter-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: flutter-app-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: flutter-app-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
```

**6. Complete Deployment Script**

**deploy.sh:**
```bash
#!/bin/bash

set -e

# Variables
IMAGE_NAME="yourusername/flutter-app"
VERSION="1.0"
NAMESPACE="production"

echo "Building Docker image..."
docker build -t $IMAGE_NAME:$VERSION .

echo "Pushing to registry..."
docker push $IMAGE_NAME:$VERSION

echo "Creating namespace if not exists..."
kubectl create namespace $NAMESPACE --dry-run=client -o yaml | kubectl apply -f -

echo "Applying ConfigMap..."
kubectl apply -f k8s/configmap.yaml -n $NAMESPACE

echo "Applying Secret..."
kubectl apply -f k8s/secret.yaml -n $NAMESPACE

echo "Applying Deployment..."
kubectl apply -f k8s/deployment.yaml -n $NAMESPACE

echo "Applying Service..."
kubectl apply -f k8s/service.yaml -n $NAMESPACE

echo "Applying Ingress..."
kubectl apply -f k8s/ingress.yaml -n $NAMESPACE

echo "Applying HPA..."
kubectl apply -f k8s/hpa.yaml -n $NAMESPACE

echo "Waiting for rollout..."
kubectl rollout status deployment/flutter-app -n $NAMESPACE

echo "Deployment complete!"
kubectl get pods -n $NAMESPACE
kubectl get services -n $NAMESPACE
kubectl get ingress -n $NAMESPACE
```

### 3.7 Advanced Kubernetes Concepts

**1. StatefulSets (for databases)**

**statefulset.yaml:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

**2. DaemonSet (runs on every node)**

**daemonset.yaml:**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
```

**3. Jobs and CronJobs**

**cronjob.yaml:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
spec:
  schedule: "0 2 * * *"  # Every day at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:15
            command:
            - /bin/sh
            - -c
            - pg_dump -h postgres -U user mydb > /backup/backup-$(date +%Y%m%d).sql
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          restartPolicy: OnFailure
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pvc
```

**4. Network Policies**

**networkpolicy.yaml:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: flutter-app-network-policy
spec:
  podSelector:
    matchLabels:
      app: flutter-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: production
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 8080
```

### 3.8 Monitoring and Logging

**1. Install Metrics Server**

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# View metrics
kubectl top nodes
kubectl top pods
```

**2. Prometheus & Grafana (Monitoring)**

```bash
# Install using Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack

# Access Grafana
kubectl port-forward svc/prometheus-grafana 3000:80
# Open http://localhost:3000 (admin/prom-operator)
```

**3. ELK Stack (Logging)**

```yaml
# elasticsearch.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
spec:
  serviceName: elasticsearch
  replicas: 1
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
        ports:
        - containerPort: 9200
        env:
        - name: discovery.type
          value: single-node
```

### 3.9 Kubernetes Best Practices

**1. Resource Management**
```yaml
# Always set resource limits
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

**2. Health Checks**
```yaml
# Implement both liveness and readiness probes
livenessProbe:
  httpGet:
    path: /health
    port: 80
readinessProbe:
  httpGet:
    path: /ready
    port: 80
```

**3. Use Namespaces**
```bash
# Separate environments
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace production
```

**4. Rolling Updates**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0  # Zero downtime
```

**5. Pod Disruption Budgets**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: flutter-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: flutter-app
```

---

## 4. CI/CD Integration

### 4.1 GitHub Actions

**.github/workflows/deploy.yml:**
```yaml
name: Build and Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  DOCKER_IMAGE: yourusername/flutter-app
  K8S_NAMESPACE: production

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Analyze code
        run: flutter analyze

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            ${{ env.DOCKER_IMAGE }}:latest
            ${{ env.DOCKER_IMAGE }}:${{ github.sha }}
          cache-from: type=registry,ref=${{ env.DOCKER_IMAGE }}:buildcache
          cache-to: type=registry,ref=${{ env.DOCKER_IMAGE }}:buildcache,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3

      - name: Setup kubectl
        uses: azure/setup-kubectl@v3

      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > kubeconfig
          export KUBECONFIG=./kubeconfig

      - name: Update deployment
        run: |
          kubectl set image deployment/flutter-app \
            flutter-app=${{ env.DOCKER_IMAGE }}:${{ github.sha }} \
            -n ${{ env.K8S_NAMESPACE }}

      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/flutter-app \
            -n ${{ env.K8S_NAMESPACE }}

      - name: Verify deployment
        run: |
          kubectl get pods -n ${{ env.K8S_NAMESPACE }}
          kubectl get services -n ${{ env.K8S_NAMESPACE }}
```

### 4.2 GitLab CI

**.gitlab-ci.yml:**
```yaml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_IMAGE: registry.gitlab.com/yourusername/flutter-app
  K8S_NAMESPACE: production

test:
  stage: test
  image: ghcr.io/cirruslabs/flutter:stable
  script:
    - flutter pub get
    - flutter test
    - flutter analyze

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $DOCKER_IMAGE:$CI_COMMIT_SHA .
    - docker tag $DOCKER_IMAGE:$CI_COMMIT_SHA $DOCKER_IMAGE:latest
    - docker push $DOCKER_IMAGE:$CI_COMMIT_SHA
    - docker push $DOCKER_IMAGE:latest
  only:
    - main

deploy:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context $KUBE_CONTEXT
    - kubectl set image deployment/flutter-app flutter-app=$DOCKER_IMAGE:$CI_COMMIT_SHA -n $K8S_NAMESPACE
    - kubectl rollout status deployment/flutter-app -n $K8S_NAMESPACE
  only:
    - main
```

---

## 5. Production Best Practices

### 5.1 Security

**1. Use Non-Root Containers**
```dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
```

**2. Scan Images for Vulnerabilities**
```bash
# Using Trivy
docker run aquasec/trivy image yourusername/flutter-app:1.0

# Using Snyk
snyk container test yourusername/flutter-app:1.0
```

**3. Use Secrets Management**
```bash
# Never hardcode secrets
# Use Kubernetes secrets or external tools like:
# - HashiCorp Vault
# - AWS Secrets Manager
# - Google Secret Manager
# - Azure Key Vault
```

**4. Enable RBAC in Kubernetes**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

### 5.2 Performance

**1. Optimize Docker Images**
```dockerfile
# Use Alpine images
FROM nginx:alpine

# Multi-stage builds
FROM ubuntu:22.04 AS build
# ... build steps
FROM nginx:alpine
COPY --from=build /app/build/web /usr/share/nginx/html
```

**2. Enable Caching**
```yaml
# In Kubernetes
volumes:
- name: cache
  emptyDir: {}
```

**3. Use CDN for Static Assets**
```dart
// In Flutter
const String cdnUrl = 'https://cdn.example.com';
Image.network('$cdnUrl/images/logo.png');
```

### 5.3 Monitoring and Alerting

**1. Application Performance Monitoring (APM)**
```yaml
# Sentry integration in Flutter
dependencies:
  sentry_flutter: ^7.0.0

# main.dart
await SentryFlutter.init(
  (options) {
    options.dsn = 'your-sentry-dsn';
  },
  appRunner: () => runApp(MyApp()),
);
```

**2. Set Up Alerts**
```yaml
# Prometheus AlertManager rules
groups:
- name: flutter-app
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
    annotations:
      summary: "High error rate detected"
```

### 5.4 Backup and Disaster Recovery

```bash
# Backup Kubernetes resources
kubectl get all --all-namespaces -o yaml > cluster-backup.yaml

# Backup using Velero
velero backup create my-backup --include-namespaces production

# Restore
velero restore create --from-backup my-backup
```

### 5.5 Cost Optimization

**1. Right-size Resources**
```yaml
# Start conservative, monitor, then adjust
resources:
  requests:
    memory: "64Mi"
    cpu: "50m"
```

**2. Use Spot Instances (AWS)**
```yaml
# EKS node group with spot instances
nodeSelector:
  kubernetes.io/lifecycle: spot
```

**3. Enable Cluster Autoscaler**
```bash
helm install cluster-autoscaler autoscaler/cluster-autoscaler \
  --set autoDiscovery.clusterName=my-cluster
```

---

## Summary

### Deployment Journey

1. **Manual Deployment** - Understand the basics
2. **Docker** - Package your app consistently
3. **Kubernetes** - Scale and manage in production
4. **CI/CD** - Automate everything
5. **Monitor & Optimize** - Keep improving

### Quick Command Reference

```bash
# Docker
docker build -t my-app .
docker run -d -p 8080:80 my-app
docker-compose up -d

# Kubernetes
kubectl apply -f deployment.yaml
kubectl get pods
kubectl logs pod-name
kubectl scale deployment my-app --replicas=5

# Monitoring
kubectl top nodes
kubectl top pods
kubectl describe pod pod-name
```

### Next Steps

1. Practice with Minikube locally
2. Deploy a simple app to production
3. Set up monitoring and logging
4. Implement CI/CD pipeline
5. Learn advanced topics (service mesh, GitOps, etc.)

---

## Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Flutter Deployment Guide](https://flutter.dev/docs/deployment)
- [CNCF Landscape](https://landscape.cncf.io/)
- [12 Factor App](https://12factor.net/)

**Ready to deploy?** Start with Docker locally, then progress to Kubernetes when you're comfortable with containerization!
