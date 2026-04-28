# RxPulse: Pharmacy Management Platform

## Capstone Project Documentation

**Project Start Date:** 2024  
**Current Version:** 1.0.0  
**Project Status:** Production-Ready Capstone Project

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Application Architecture](#2-application-architecture)
3. [Kubernetes & Infrastructure Details](#3-kubernetes--infrastructure-details)
4. [CI/CD Pipeline Details](#4-cicd-pipeline-details)
5. [Security Implementations](#5-security-implementations)
6. [Monitoring & Observability](#6-monitoring--observability)
7. [Repository Structure](#7-repository-structure)
8. [Environment Details](#8-environment-details)
9. [Deployment Guide](#9-deployment-guide)
10. [Architecture Diagrams](#10-architecture-diagrams)

---

## 1. PROJECT OVERVIEW

### Project Name

**RxPulse** — Pharmacy Management Platform

### Core Purpose

RxPulse is a comprehensive pharmacy management system designed to streamline medicine catalog management, inventory tracking, stock alerts, and user authentication. The platform provides a modern web interface for pharmacists and managers to oversee pharmaceutical operations with real-time stock monitoring and reporting capabilities.

### Problem Statement

Pharmacies face challenges in:

- Manual inventory management leading to stockouts and overstocking
- Lack of real-time visibility into medicine expiration dates
- Fragmented systems for user authentication and authorization
- Difficulty tracking medicine movements and generating inventory reports
- Manual catalog maintenance for thousands of medicines

### Solution Provided

RxPulse solves these issues through:

- Centralized microservices-based architecture for scalability
- Real-time inventory tracking with automated alerts
- JWT-based authentication with role-based access control
- Comprehensive medicine catalog with search and filtering
- Stock movement tracking and reporting capabilities
- Cloud-native Kubernetes deployment for high availability

### Tech Stack

#### **Frontend**

- **Framework:** React 18.3.1
- **Build Tool:** Vite 5.2.11
- **Styling:** Tailwind CSS 3.4.3
- **Routing:** React Router DOM 6.23.1
- **HTTP Client:** Axios 1.6.8
- **Charting:** Recharts 2.12.7
- **UI Icons:** Lucide React 0.378.0
- **Node Version:** 20 (Alpine)
- **Package Manager:** npm

#### **Backend Services**

- **Runtime:** Node.js 18 (Alpine)
- **Framework:** Express.js 4.18.3 - 4.19.2
- **Database:** MongoDB 6.0
- **Authentication:** JWT (jsonwebtoken 9.0.2)
- **Password Hashing:** bcryptjs 2.4.3
- **HTTP Client:** Axios 1.6.8 (for inter-service communication)
- **CORS:** Express CORS 2.8.5
- **Environment Management:** dotenv 16.4.5
- **Development:** Nodemon 3.1.0

#### **Infrastructure & DevOps**

- **Container Orchestration:** Kubernetes
- **Ingress:** KGateway (Kubernetes Gateway API)
- **Package Manager:** Helm 3.x
- **Deployment Automation:** ArgoCD (GitOps)
- **Progressive Delivery:** Argo Rollouts with Blue-Green Strategy
- **CI/CD:** GitHub Actions
- **Container Registry:** Docker Hub
- **Image Scanning:** Trivy
- **Code Quality:** SonarQube / SonarCloud
- **Vulnerability Scanning:** Snyk
- **Infrastructure as Code:** Helm Charts, YAML Manifests

#### **Databases**

- **Primary Database:** MongoDB 6.0
- **Database Type:** NoSQL (Document-based)
- **Persistence:** PersistentVolumeClaims with NFS storage class
- **Per-Service Databases:** Separate MongoDB instances for each microservice (User, Catalog, Inventory)

---

## 2. APPLICATION ARCHITECTURE

### 2.1 Microservices Overview

The RxPulse platform is built on a **microservices architecture** with 4 independent backend services and 1 React frontend.

#### **Service 1: User Service (Port 3001)**

| Property                 | Value               |
| ------------------------ | ------------------- |
| **Service Name**         | user-service        |
| **Repository**           | rxpulse-userservice |
| **Main File**            | src/app.js          |
| **Port**                 | 3001                |
| **Container**            | node:18-alpine      |
| **Database**             | MongoDB (userdb)    |
| **Replicas (Dev)**       | 1                   |
| **Replicas (Prod)**      | 1                   |
| **Image Version (Dev)**  | develop-8f27dcd3    |
| **Image Version (Prod)** | user-v0.0.2         |

**Responsibilities:**

- User registration and account creation
- User authentication with JWT token generation
- User profile management (get/update)
- JWT token verification
- Role-based user management
- Seed data initialization for test users

**Key Dependencies:**

- express (4.19.2)
- mongoose (8.3.4)
- jsonwebtoken (9.0.2)
- bcryptjs (2.4.3)
- cors (2.8.5)

**API Endpoints:**

```
POST   /api/users/register          — Register new user
POST   /api/users/login             — Authenticate and get JWT token
GET    /api/users/profile           — Get current user profile (protected)
PUT    /api/users/profile           — Update user profile (protected)
GET    /api/users/verify            — Verify JWT token (protected)
GET    /health                      — Health check endpoint
```

**Database Models:**

- User (email, password, firstName, lastName, role, createdAt, updatedAt)

**User Roles Defined:**

- admin
- manager
- pharmacist
- customer

---

#### **Service 2: Catalog Service (Port 3002)**

| Property                 | Value                  |
| ------------------------ | ---------------------- |
| **Service Name**         | catalog-service        |
| **Repository**           | rxpulse-catalogservice |
| **Main File**            | src/app.js             |
| **Port**                 | 3002                   |
| **Container**            | node:18-alpine         |
| **Database**             | MongoDB (catalogdb)    |
| **Replicas (Dev)**       | 1                      |
| **Replicas (Prod)**      | 1                      |
| **Image Version (Dev)**  | develop-3f69a4aa       |
| **Image Version (Prod)** | catalog-v0.0.2         |

**Responsibilities:**

- Manage medicine catalog
- Organize medicines by categories
- Search and filter medicines
- Track medicine expiration dates
- Perform CRUD operations on medicines and categories
- Seed catalog data

**Key Dependencies:**

- express (4.18.3)
- mongoose (8.2.4)
- jsonwebtoken (9.0.2)
- cors (2.8.5)

**API Endpoints:**

```
GET    /api/catalog/medicines              — Get all medicines (protected)
GET    /api/catalog/medicines/:id          — Get medicine by ID (protected)
POST   /api/catalog/medicines              — Create medicine (protected, manager/admin only)
PUT    /api/catalog/medicines/:id          — Update medicine (protected, manager/admin only)
DELETE /api/catalog/medicines/:id          — Delete medicine (protected, admin only)
GET    /api/catalog/medicines/search       — Search medicines (protected)
GET    /api/catalog/medicines/expiring     — Get expiring medicines (protected)
GET    /api/catalog/categories             — Get all categories (protected)
POST   /api/catalog/categories             — Create category (protected, manager/admin only)
GET    /health                             — Health check endpoint
```

**Database Models:**

- Medicine (name, description, category, manufacturer, expiryDate, price, quantity, createdAt, updatedAt)
- Category (name, description, createdAt, updatedAt)

---

#### **Service 3: Inventory Service (Port 3003)**

| Property                 | Value                    |
| ------------------------ | ------------------------ |
| **Service Name**         | inventory-service        |
| **Repository**           | rxpulse-inventoryservice |
| **Main File**            | src/app.js               |
| **Port**                 | 3003                     |
| **Container**            | node:18-alpine           |
| **Database**             | MongoDB (inventorydb)    |
| **Replicas (Dev)**       | 1                        |
| **Replicas (Prod)**      | 1                        |
| **Image Version (Dev)**  | develop-5b9aaaf4         |
| **Image Version (Prod)** | inventory-v0.0.2         |

**Responsibilities:**

- Track real-time stock levels
- Record stock movements (inbound/outbound)
- Generate low-stock alerts
- Manage alert thresholds per medicine
- Generate inventory reports and statistics
- Track stock history and movements

**Key Dependencies:**

- express (4.18.3)
- mongoose (8.2.4)
- jsonwebtoken (9.0.2)
- axios (1.6.8) — for inter-service communication
- cors (2.8.5)

**API Endpoints:**

```
GET    /api/inventory/stocks                    — Get all stocks (protected)
GET    /api/inventory/stocks/:medicineId        — Get stock for medicine (protected)
POST   /api/inventory/stocks/stock-in           — Record stock inbound (protected, pharmacist/manager/admin)
POST   /api/inventory/stocks/stock-out          — Record stock outbound (protected, pharmacist/manager/admin)
PUT    /api/inventory/stocks/:id/threshold      — Update alert threshold (protected, manager/admin)
GET    /api/inventory/movements                 — Get all movements (protected)
GET    /api/inventory/alerts                    — Get all alerts (protected)
GET    /api/inventory/reports                   — Get reports and statistics (protected)
GET    /health                                  — Health check endpoint
```

**Database Models:**

- Stock (medicineId, quantity, lastUpdated, threshold, createdAt, updatedAt)
- Movement (medicineId, type, quantity, reason, recordedBy, timestamp, createdAt)
- Alert (medicineId, alertType, threshold, quantity, createdAt)

---

#### **Service 4: Frontend Service (React SPA)**

| Property                 | Value                                       |
| ------------------------ | ------------------------------------------- |
| **Service Name**         | frontend                                    |
| **Repository**           | rxpulse-frontend                            |
| **Technology**           | React 18.3.1 + Vite 5.2.11                  |
| **Port**                 | 8080                                        |
| **Container**            | nginx:1.27-alpine                           |
| **Build Stage**          | node:20-alpine                              |
| **Replicas (Dev)**       | 2                                           |
| **Replicas (Prod)**      | 2                                           |
| **Image Version (Dev)**  | develop-e4ff3d5f                            |
| **Image Version (Prod)** | frontend-v0.0.3                             |
| **Deployment Strategy**  | Argo Rollouts Blue-Green (Manual Promotion) |
| **HPA (Dev)**            | Enabled: min=2, max=5                       |
| **HPA (Prod)**           | Enabled: min=2, max=10                      |

**Responsibilities:**

- Provide web-based user interface
- User registration and login
- Medicine shopping and browsing
- Inventory management dashboard
- Stock alerts and reports visualization
- Admin panels for system management
- Role-based UI rendering

**Key Features:**

- React Router DOM for client-side routing
- Recharts for data visualization
- Tailwind CSS for responsive styling
- Axios for API communication
- JWT token management in localStorage
- Real-time component updates

**Directory Structure:**

```
src/
├── App.jsx                      — Main React component
├── index.css                    — Global styles
├── main.jsx                     — Entry point
├── api/
│   ├── authApi.js              — Auth API calls
│   ├── catalogApi.js           — Catalog API calls
│   ├── inventoryApi.js         — Inventory API calls
│   └── userApi.js              — User API calls
├── components/
│   ├── admin/                  — Admin dashboard components
│   ├── charts/                 — Data visualization components
│   ├── common/                 — Reusable components
│   ├── inventory/              — Inventory management components
│   └── medicines/              — Medicine browser components
├── context/
│   ├── AuthContext.jsx         — Authentication state
│   └── CartContext.jsx         — Shopping cart state
└── pages/
    ├── Cart.jsx
    ├── Home.jsx
    ├── Login.jsx
    ├── MedicineDetail.jsx
    ├── Register.jsx
    ├── Shop.jsx
    ├── admin/                  — Admin pages
    ├── alerts/                 — Alert pages
    ├── auth/                   — Auth pages
    ├── dashboard/              — Dashboard pages
    ├── inventory/              — Inventory pages
    ├── medicines/              — Medicine management pages
    ├── movements/              — Movement tracking pages
    └── reports/                — Report pages
```

**Build Configuration:**

- Vite Config: vite.config.js
- Tailwind Config: tailwind.config.js
- PostCSS Config: postcss.config.js
- Nginx Config: nginx.conf (production server)

---

### 2.2 Service Communication

#### **Communication Protocol**

- **Type:** REST API over HTTP
- **Data Format:** JSON
- **Authentication:** JWT Bearer tokens

#### **Service Interactions**

```
Frontend (8080)
  ├─→ User Service (3001)      — Auth, profile management
  ├─→ Catalog Service (3002)   — Medicine catalog, search
  └─→ Inventory Service (3003) — Stock, alerts, reports

Inventory Service
  └─→ Catalog Service (3002)   — Fetch medicine details for stock tracking

Services → MongoDB Services
  ├─→ mongo-user (27017)       — User database
  ├─→ mongo-catalog (27017)    — Catalog database
  └─→ mongo-inventory (27017)  — Inventory database
```

#### **Inter-Service Call Examples**

- Inventory Service calls Catalog Service to enrich stock data with medicine details
- Frontend calls all three services for different operations based on user actions

#### **Authentication Flow**

1. User posts credentials to `/api/users/login`
2. User Service validates and returns JWT token
3. Frontend stores JWT in localStorage
4. All subsequent requests include `Authorization: Bearer <JWT>`
5. Services validate JWT via authMiddleware

#### **External APIs & Integrations**

- **None currently configured** — Platform is self-contained
- **Email Notifications:** CI/CD pipeline sends SMTP notifications (configured via secrets)
- **Code Quality Services:** SonarQube/SonarCloud, Snyk (external scanning)
- **Container Registry:** Docker Hub (image storage)

---

## 3. KUBERNETES & INFRASTRUCTURE DETAILS

### 3.1 Cluster Setup

| Property                | Value                                   |
| ----------------------- | --------------------------------------- |
| **Cluster Type**        | Single cluster (local Kubernetes)       |
| **Architecture**        | On-premises or local development        |
| **Kubernetes Version**  | 1.24+ (inferred from API versions used) |
| **Gateway API Version** | v1 (KGateway, Kubernetes native)        |
| **Helm Version**        | 3.x                                     |
| **Namespaces**          | rxpulse-dev, rxpulse-prod               |

### 3.2 Kubernetes Manifests & Resources

#### **Namespaces**

```yaml
# Development Namespace
metadata:
  name: rxpulse-dev
  annotations:
    argocd.argoproj.io/sync-wave: "-1"

# Production Namespace
metadata:
  name: rxpulse-prod
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
```

#### **Deployments**

| Service           | Type             | Replicas (Dev) | Replicas (Prod) | Port | Container Image                          |
| ----------------- | ---------------- | -------------- | --------------- | ---- | ---------------------------------------- |
| user-service      | Deployment       | 1              | 1               | 3001 | rxpulsehub/rxpulse-user:develop-...      |
| catalog-service   | Deployment       | 1              | 1               | 3002 | rxpulsehub/rxpulse-catalog:develop-...   |
| inventory-service | Deployment       | 1              | 1               | 3003 | rxpulsehub/rxpulse-inventory:develop-... |
| **frontend**      | **Argo Rollout** | 2              | 2               | 8080 | rxpulsehub/rxpulse-frontend:develop-...  |

#### **Argo Rollouts Configuration (Frontend)**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: frontend
spec:
  replicas: 2 (dev), 2 (prod)
  strategy:
    blueGreen:
      activeService: frontend-active
      previewService: frontend-preview
      autoPromotionEnabled: false # Manual promotion required
      scaleDownDelaySeconds: 120 # Keep old version for 2 minutes
  template:
    spec:
      terminationGracePeriodSeconds: 30
      containers:
        - name: frontend
          securityContext:
            allowPrivilegeEscalation: false
            runAsNonRoot: true
            runAsUser: 1001
            runAsGroup: 1001
```

**Blue-Green Strategy Details:**

- **Blue Environment:** Currently active version serving traffic
- **Green Environment:** New version deployed but not receiving traffic
- **Preview Service:** Routes traffic to green (preview) for testing
- **Active Service:** Routes traffic to blue (production)
- **Manual Promotion:** Requires manual intervention to switch traffic from blue to green
- **Rollback:** Can quickly rollback by keeping blue running

#### **StatefulSets (MongoDB Instances)**

Three dedicated MongoDB StatefulSets, one per microservice:

```yaml
# Per-Service MongoDB StatefulSet Pattern
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo-{{ service }} # mongo-user, mongo-catalog, mongo-inventory
spec:
  replicas: 1
  serviceName: mongo-{{ service }}
  selector:
    matchLabels:
      app: mongo-{{ service }}
  template:
    spec:
      securityContext:
        fsGroup: 999
        runAsUser: 999
        runAsGroup: 999
        runAsNonRoot: true
      containers:
        - name: mongo-{{ service }}
          image: mongo:6.0
          command:
            - mongod
            - --bind_ip_all
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongo-{{ service }}-data
              mountPath: /data/db
          livenessProbe:
            exec:
              command:
                - mongosh
                - --eval
                - "db.adminCommand('ping')"
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            exec:
              command:
                - mongosh
                - --eval
                - "db.adminCommand('ping')"
            initialDelaySeconds: 15
            periodSeconds: 5
```

**MongoDB Services Details:**

| StatefulSet     | Service         | Database Name | PVC Name            | Storage (Dev) | Storage (Prod) | Port  |
| --------------- | --------------- | ------------- | ------------------- | ------------- | -------------- | ----- |
| mongo-user      | mongo-user      | userdb        | mongo-user-pvc      | 5Gi           | 20Gi           | 27017 |
| mongo-catalog   | mongo-catalog   | catalogdb     | mongo-catalog-pvc   | 5Gi           | 20Gi           | 27017 |
| mongo-inventory | mongo-inventory | inventorydb   | mongo-inventory-pvc | 5Gi           | 20Gi           | 27017 |

**MongoDB Configuration:**

- Version: 6.0
- Replicas per StatefulSet: 1
- Storage Class: nfs-client
- Resource Requests (Dev): CPU 250m, Memory 512Mi
- Resource Limits (Dev): CPU 500m, Memory 1Gi
- Resource Requests (Prod): CPU 500m, Memory 1Gi
- Resource Limits (Prod): CPU 1000m, Memory 2Gi

#### **Horizontal Pod Autoscaler (HPA)**

**Frontend HPA Configuration:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-hpa
spec:
  scaleTargetRef:
    apiVersion: argoproj.io/v1alpha1
    kind: Rollout
    name: frontend
  minReplicas: 2 (dev), 2 (prod)
  maxReplicas: 5 (dev), 10 (prod)
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

**Scaling Behavior:**

- **CPU Scaling:** When CPU utilization exceeds 70%, add replicas
- **Memory Scaling:** When memory utilization exceeds 80%, add replicas
- **Dev Environment:** Scale between 2-5 replicas
- **Prod Environment:** Scale between 2-10 replicas

---

### 3.3 Network Configuration

#### **KGateway (Kubernetes Gateway API)**

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: rxpulse-gateway
spec:
  gatewayClassName: standard # Native K8s gateway class
  listeners:
    - name: http
      port: 80
      protocol: HTTP
      allowedRoutes:
        namespaces:
          from: Same # Only same namespace routes
```

#### **HTTPRoute Configuration**

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: rxpulse-routes
spec:
  parentRefs:
    - name: rxpulse-gateway
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api/users/
      backendRefs:
        - name: user-service
          port: 3001

    - matches:
        - path:
            type: PathPrefix
            value: /api/catalog/
      backendRefs:
        - name: catalog-service
          port: 3002

    - matches:
        - path:
            type: PathPrefix
            value: /api/inventory/
      backendRefs:
        - name: inventory-service
          port: 3003

    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: frontend-active
          port: 8080
```

**Routing Rules:**

- `/api/users/*` → user-service:3001
- `/api/catalog/*` → catalog-service:3002
- `/api/inventory/*` → inventory-service:3003
- `/*` (root) → frontend-active:8080 (Argo Rollout blue-green service)

#### **Network Policies**

Example: Catalog Service Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-catalog-to-mongo
spec:
  podSelector:
    matchLabels:
      app: mongo-catalog
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - catalog-service
                  - catalog-seed
```

**Network Policy Strategy:**

- Each MongoDB instance only accepts traffic from its corresponding service
- Database pods are isolated from other services
- Prevents data leakage across service boundaries
- Services communicate via defined HTTP routes

---

### 3.4 Services

**Service Types:**

| Service           | Type      | Port  | Target Port | Selector               |
| ----------------- | --------- | ----- | ----------- | ---------------------- |
| user-service      | ClusterIP | 3001  | 3001        | app: user-service      |
| catalog-service   | ClusterIP | 3002  | 3002        | app: catalog-service   |
| inventory-service | ClusterIP | 3003  | 3003        | app: inventory-service |
| frontend-active   | ClusterIP | 8080  | 8080        | app: frontend (blue)   |
| frontend-preview  | ClusterIP | 8080  | 8080        | app: frontend (green)  |
| mongo-user        | ClusterIP | 27017 | 27017       | app: mongo-user        |
| mongo-catalog     | ClusterIP | 27017 | 27017       | app: mongo-catalog     |
| mongo-inventory   | ClusterIP | 27017 | 27017       | app: mongo-inventory   |

**All services use ClusterIP type** — internal communication within cluster.

---

### 3.5 Resource Limits & Requests

#### **Backend Services (User, Catalog, Inventory)**

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 300m
    memory: 256Mi
```

#### **Frontend Service**

**Dev:**

```yaml
resources:
  requests:
    cpu: 100m (default)
    memory: 128Mi (default)
  limits:
    cpu: 300m (default)
    memory: 256Mi (default)
```

**Prod:**

```yaml
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 1000m
    memory: 1Gi
```

#### **MongoDB Services**

**Dev:**

```yaml
resources:
  requests:
    cpu: 250m
    memory: 512Mi
  limits:
    cpu: 500m
    memory: 1Gi
```

**Prod:**

```yaml
resources:
  requests:
    cpu: 500m
    memory: 1Gi
  limits:
    cpu: 1000m
    memory: 2Gi
```

---

### 3.6 ConfigMaps & Secrets

#### **ConfigMaps**

**rxpulse-config** — Node environment and service port configuration:

```yaml
CATALOG_SERVICE_PORT: 3002
INVENTORY_SERVICE_PORT: 3003
USER_SERVICE_PORT: 3001
NODE_ENV: development (dev) | production (prod)
```

**Values per environment:**

- Dev: NODE_ENV=development
- Prod: NODE_ENV=production

#### **Secrets**

**rxpulse-secrets** — Sensitive data:

```yaml
MONGO_ROOT_USERNAME: (from values)
MONGO_ROOT_PASSWORD: (from values)
MONGO_USER_URI: mongodb://...
MONGO_CATALOG_URI: mongodb://...
MONGO_INVENTORY_URI: mongodb://...
JWT_SECRET: (from CI/CD secrets)
```

All services reference these secrets via `envFrom: secretRef`

#### **SealedSecrets (Prod)**

Production uses Kubernetes SealedSecrets for encrypted secrets at rest:

- `environments/prod/sealed-secret.yaml` — Encrypted secret manifest
- Sealed with cluster-specific key
- Automatically unsealed by SealedSecrets controller

---

### 3.7 Storage

#### **PersistentVolumeClaims (PVCs)**

| PVC Name            | Service         | Size (Dev) | Size (Prod) | Storage Class | Access Mode   |
| ------------------- | --------------- | ---------- | ----------- | ------------- | ------------- |
| mongo-user-pvc      | mongo-user      | 5Gi        | 20Gi        | nfs-client    | ReadWriteOnce |
| mongo-catalog-pvc   | mongo-catalog   | 5Gi        | 20Gi        | nfs-client    | ReadWriteOnce |
| mongo-inventory-pvc | mongo-inventory | 5Gi        | 20Gi        | nfs-client    | ReadWriteOnce |

**Storage Class:**

- Type: nfs-client (NFS provisioner)
- Allows dynamic provisioning of NFS volumes
- ReadWriteOnce access (single writer)

---

### 3.8 Health Probes

#### **Liveness Probes** (Restart container if unhealthy)

**Backend Services:**

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 3001 (respective port)
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
```

**MongoDB:**

```yaml
livenessProbe:
  exec:
    command:
      - mongosh
      - --eval
      - "db.adminCommand('ping')"
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
```

#### **Readiness Probes** (Remove from service if not ready)

**Backend Services:**

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 3001 (respective port)
  initialDelaySeconds: 15
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
```

**MongoDB:**

```yaml
readinessProbe:
  exec:
    command:
      - mongosh
      - --eval
      - "db.adminCommand('ping')"
  initialDelaySeconds: 15
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
```

---

## 4. CI/CD PIPELINE DETAILS

### 4.1 Overview

**CI/CD Platform:** GitHub Actions  
**Trigger:** Push to branches or Pull Requests  
**Branching Strategy:** Git Flow (main, develop, feature/_, release/_, hotfix/\*)  
**Artifact Storage:** Docker Hub  
**Deployment Orchestration:** ArgoCD

---

### 4.2 GitHub Actions Workflows

#### **Workflow File Location**

```
rxpulse-ci-workflows/.github/workflows/
├── build-image.yaml          — Docker image build and semantic versioning
├── code-scan.yaml            — SonarQube + Snyk scanning
├── push-image.yaml           — Push to Docker Hub
├── update-helm.yaml          — Update Helm values with new image tags
└── notify.yaml               — Email notifications on pipeline completion
```

#### **Workflow 1: code-scan.yaml**

**Purpose:** Static code analysis and vulnerability scanning

**Triggers:**

- Called from main workflows via `workflow_call`
- Triggered on every commit to any branch

**Inputs:**

- `service-name` — Microservice name (frontend, user, catalog, inventory)
- `node-version` — Node.js version (default: 18)

**Secrets Required:**

- `SONAR_TOKEN` — SonarCloud authentication
- `SONAR_HOST_URL` — SonarCloud server URL (optional)
- `SNYK_TOKEN` — Snyk authentication

**Jobs:**

| Job           | Step              | Tool        | Action                              |
| ------------- | ----------------- | ----------- | ----------------------------------- |
| **sonarqube** | 1. Checkout       | Git         | Fetch source code                   |
|               | 2. Setup Node     | Actions     | Install Node.js 18                  |
|               | 3. Install deps   | npm         | npm ci                              |
|               | 4. SonarQube scan | SonarSource | Run analysis                        |
|               | 5. Quality Gate   | SonarSource | Check quality gate (timeout: 5 min) |
| **snyk**      | 1. Checkout       | Git         | Fetch source code                   |
|               | 2. Setup Node     | Actions     | Install Node.js 18                  |
|               | 3. Install deps   | npm         | npm install                         |
|               | 4. Setup Snyk     | Snyk        | Install Snyk CLI                    |
|               | 5. Snyk scan      | Snyk        | Security scan (severity: high)      |
|               | 6. Convert report | npm         | Generate HTML report                |

**SonarQube Configuration:**

- Project Key: `rxpulse-{service-name}`
- Project Name: `rxpulse-{service-name}`
- Source: `src/`
- Quality Gate: Must pass before proceeding

**Snyk Configuration:**

- Severity Threshold: HIGH (fail on high/critical vulnerabilities)
- Output: JSON report + HTML conversion
- Dependency Scanning: package.json

**Outputs:**

- sonarqube_result: success/failure
- snyk_result: success/failure

---

#### **Workflow 2: build-image.yaml**

**Purpose:** Build Docker image with semantic versioning and image scanning

**Triggers:**

- Called from main workflows via `workflow_call`
- Triggered on push to any branch

**Inputs:**

- `service-name` — Microservice name
- `service-path` — Build context path (default: ".")
- `run-trivy` — Run Trivy scanner (true on PR, false on push)

**Secrets:**

- `DOCKERHUB_USERNAME` — Docker Hub user
- `DOCKERHUB_TOKEN` — Docker Hub token

**Jobs:**

| Job              | Purpose                    | Logic                                                  |
| ---------------- | -------------------------- | ------------------------------------------------------ |
| **generate-tag** | Calculate semantic version | If main: semantic versioning; else: branch-sha tagging |
| **build-image**  | Build Docker image         | Multi-stage build, push to artifact storage            |
| **trivy**        | Scan image for CVEs        | Container image scanning                               |

**Versioning Strategy:**

**On main branch:**

```
Semantic Versioning:
  v1.2.3 → v1.2.4 (patch bump)
  Uses mathieudutour/github-tag-action@v6.2
  Default bump: patch
  Tag prefix: v
```

**On other branches:**

```
Branch-Commit Tagging:
  develop → develop-a1b2c3d4 (branch slug + short SHA)
  feature/auth → feature-auth-a1b2c3d4
  Replaces "/" with "-" in branch names
```

**Trivy Configuration:**

- Scans built Docker image for vulnerabilities
- Fails on CRITICAL/HIGH CVEs (by default)
- Generates report for review

**Outputs:**

- image-tag: Computed tag (e.g., v1.2.4 or develop-a1b2c3d4)
- tag-exists: Boolean if git tag exists
- image-exists: Boolean if Docker image exists
- build-result: success/failure
- trivy-result: success/failure

---

#### **Workflow 3: push-image.yaml**

**Purpose:** Push Docker image to Docker Hub and create git tags

**Triggers:**

- Called after successful build-image workflow

**Inputs:**

- `service-name` — Microservice name
- `image-tag` — Tag from build-image output
- `tag-exists` — Whether git tag already exists

**Secrets:**

- `DOCKERHUB_USERNAME` — Docker Hub user
- `DOCKERHUB_TOKEN` — Docker Hub token

**Jobs:**

| Step                       | Action                                  |
| -------------------------- | --------------------------------------- |
| Download artifact          | Download docker-image-rxpulse-{service} |
| Load image                 | `docker load -i /tmp/image.tar`         |
| Login Docker Hub           | Authenticate with credentials           |
| Tag versioned image        | `{user}/rxpulse-{service}:{tag}`        |
| Push versioned image       | Push to Docker Hub                      |
| Tag latest (main only)     | `{user}/rxpulse-{service}:latest`       |
| Push latest (main only)    | Push latest tag                         |
| Create git tag (main only) | `git tag {tag}` → push to origin        |

**Registry Details:**

- **Registry:** Docker Hub
- **Repository Format:** `rxpulsehub/rxpulse-{service}`
- **Image Tags Example:**
  - `rxpulsehub/rxpulse-frontend:develop-e4ff3d5f` (feature branch)
  - `rxpulsehub/rxpulse-frontend:v1.2.3` (main branch)
  - `rxpulsehub/rxpulse-frontend:latest` (main branch only)

**Outputs:**

- push-result: success/failure
- tag: The image tag pushed

---

#### **Workflow 4: update-helm.yaml**

**Purpose:** Update Helm values files with new image tags for GitOps deployment

**Triggers:**

- Called after successful push-image workflow

**Inputs:**

- `service-name` — Helm service key (catalog, user, inventory, frontend)
- `image-tag` — New image tag to update
- `target-branch` — develop (for dev) or main (for prod)

**Secrets:**

- `HELM_REPO_TOKEN` — PAT with contents:write on rxpulse-helmcharts
- `HELM_REPO_OWNER` — GitHub org/user owning rxpulse-helmcharts

**Jobs:**

| Step                 | Action                                                                        |
| -------------------- | ----------------------------------------------------------------------------- |
| Checkout helmcharts  | Checkout rxpulse-helmcharts repo at target branch                             |
| Select values file   | Determine file: environments/dev/values.yaml or environments/prod/values.yaml |
| Validate file exists | Check file exists, show before state                                          |
| Update image tag     | AWK script updates `global.services.{service}.tag`                            |
| Commit changes       | git add, git commit with message                                              |
| Push changes         | git push to target branch                                                     |

**Values File Structure:**

```yaml
global:
  services:
    frontend:
      tag: develop-e4ff3d5f # ← Updated here
      replicaCount: 2
    user:
      tag: develop-8f27dcd3 # ← Updated here
      replicaCount: 1
    # ... etc
```

**Branch Mapping:**

- Develop branch → environments/dev/values.yaml
- Main branch → environments/prod/values.yaml

**Commit Message Format:**

```
chore({service}): update image tag to {tag}
```

Example: `chore(frontend): update image tag to v1.2.3`

---

#### **Workflow 5: notify.yaml**

**Purpose:** Send email notifications on CI/CD completion

**Triggers:**

- Called at end of main workflows via `workflow_call`
- Always runs (if: always())

**Inputs:**

- `service-name` — Microservice name
- `branch` — Git branch name
- `image-tag` — Built image tag
- `actor` — GitHub actor who triggered
- `commit-message` — Git commit message
- `commit-sha` — Git commit SHA
- `run-url` — GitHub Actions run URL
- Result inputs: sonar-snyk, build, trivy, push, helm-dev, approval, helm-prod

**Secrets Required:**

- `SMTP_HOST` — SMTP server hostname
- `SMTP_PORT` — SMTP port (e.g., 587)
- `SMTP_USERNAME` — Email account username
- `SMTP_PASSWORD` — Email account password
- `NOTIFY_FROM_EMAIL` — Sender email
- `NOTIFY_TO_EMAIL` — Recipient email

**Email Content:**

```
Subject: [RxPulse CI/CD] SUCCESS — rxpulse-{service} @ {branch}

Service:         rxpulse-{service}
Branch:          develop
Image Tag:       develop-a1b2c3d4
Actor:           john-doe
Commit SHA:      a1b2c3d4e5f6...
Commit Message:  feat: add auth middleware

Stage Results:
- SonarCloud & Snyk    : success
- Build Image          : success
- Trivy                : success
- Push                 : success
- Helm Update (Dev)    : success
- Prod Approval Gate   : skipped
- Helm Update (Prod)   : skipped
```

---

### 4.3 Complete Pipeline Flow

#### **For Feature/Develop Branches:**

```
1. Developer pushes to feature/develop branch
        ↓
2. GitHub Actions Trigger: code-scan.yaml
   - SonarQube scan
   - Snyk vulnerability scan
   - If fails: notify and stop
        ↓
3. build-image.yaml
   - Generate tag: feature-{branch}-{sha}
   - Build Docker image (multi-stage)
   - Run Trivy scan
   - Store artifact
        ↓
4. push-image.yaml
   - Push: rxpulsehub/rxpulse-{service}:feature-branch-sha
   - Do NOT push latest
   - Do NOT create git tag
        ↓
5. update-helm.yaml
   - Update: environments/dev/values.yaml
   - Commit to develop branch
   - Push to develop
        ↓
6. ArgoCD Detects Change
   - Syncs dev environment
   - Deploys new version to rxpulse-dev namespace
        ↓
7. notify.yaml
   - Send email notification
   - Report all stage results
```

#### **For Main Branch (Production Release):**

```
1. Developer creates PR → merges to main
        ↓
2. GitHub Actions Trigger: code-scan.yaml
   - SonarQube scan
   - Snyk vulnerability scan
   - If fails: notify and stop
        ↓
3. build-image.yaml
   - Semantic versioning: v1.2.3
   - Build Docker image
   - Run Trivy scan
   - Store artifact
        ↓
4. push-image.yaml
   - Push: rxpulsehub/rxpulse-{service}:v1.2.3
   - Push: rxpulsehub/rxpulse-{service}:latest
   - Create git tag: v1.2.3
   - Push tag to origin
        ↓
5. update-helm.yaml
   - Update: environments/prod/values.yaml
   - Commit to main branch
   - Push to main
        ↓
6. Manual Approval (if configured)
   - Ops team reviews in ArgoCD
   - Approves sync to prod
        ↓
7. ArgoCD Detects Change
   - Syncs prod environment
   - Performs blue-green deployment
   - Keeps old version running (blue)
   - Deploys new version (green)
   - Waits for manual promotion
        ↓
8. Operator Promotes Green → Blue
   - Switch frontend-active service to green
   - Green becomes new production
   - Old blue scaled down after delay
        ↓
9. notify.yaml
   - Send email notification
   - Report all stage results including prod deployment
```

---

### 4.4 Branching Strategy (Git Flow)

| Branch         | Purpose                 | Deploy To         | Triggers             |
| -------------- | ----------------------- | ----------------- | -------------------- |
| **main**       | Production releases     | rxpulse-prod      | Tags, releases       |
| **develop**    | Development integration | rxpulse-dev       | Merges to develop    |
| **feature/\*** | Feature development     | None (PR preview) | PR checks            |
| **release/\*** | Release preparation     | Staging           | Before merge to main |
| **hotfix/\***  | Production patches      | rxpulse-prod      | Direct to main       |

**Branch Protection Rules (main):**

- Require pull request review before merge
- Require status checks to pass (code-scan, build-image, etc.)
- Require branches to be up to date before merge
- Allow auto-merge of PRs

---

### 4.5 CI/CD Environments

#### **Development Environment (Dev)**

- **Namespace:** rxpulse-dev
- **Helm Branch:** develop
- **Values File:** environments/dev/values.yaml
- **Auto-Deploy:** On every commit to develop branch
- **Blue-Green:** Disabled (manual promotion enabled for frontend)
- **HPA:** Enabled (min=2, max=5)
- **Database Storage:** 5Gi per MongoDB
- **Notification:** On completion

#### **Production Environment (Prod)**

- **Namespace:** rxpulse-prod
- **Helm Branch:** main
- **Values File:** environments/prod/values.yaml
- **Auto-Deploy:** On every commit to main branch
- **Blue-Green:** Enabled (manual promotion required)
- **HPA:** Enabled (min=2, max=10)
- **Database Storage:** 20Gi per MongoDB
- **Manual Approval:** (Can be configured as approval gate)
- **Notification:** On completion with prod status

---

### 4.6 Secrets Management in CI/CD

**Required GitHub Secrets:**

```
Workflow Secrets:
├── DOCKERHUB_USERNAME              — Docker Hub username
├── DOCKERHUB_TOKEN                 — Docker Hub authentication token
├── SONAR_TOKEN                     — SonarCloud API token
├── SONAR_HOST_URL                  — SonarCloud server URL
├── SNYK_TOKEN                      — Snyk API token
├── HELM_REPO_TOKEN                 — GitHub PAT (contents:write)
├── HELM_REPO_OWNER                 — GitHub org/user
├── SMTP_HOST                       — SMTP server hostname
├── SMTP_PORT                       — SMTP port number
├── SMTP_USERNAME                   — SMTP username
├── SMTP_PASSWORD                   — SMTP password
├── NOTIFY_FROM_EMAIL               — From email address
└── NOTIFY_TO_EMAIL                 — To email address
```

**Kubernetes Secrets (rxpulse-secrets):**

```
Cluster Secrets:
├── MONGO_ROOT_USERNAME             — MongoDB root username
├── MONGO_ROOT_PASSWORD             — MongoDB root password
├── MONGO_USER_URI                  — User service MongoDB connection string
├── MONGO_CATALOG_URI               — Catalog service MongoDB connection string
├── MONGO_INVENTORY_URI             — Inventory service MongoDB connection string
└── JWT_SECRET                      — JWT signing secret
```

---

## 5. SECURITY IMPLEMENTATIONS

### 5.1 Authentication & Authorization

#### **JWT-Based Authentication**

**Implementation:**

- User Service generates JWT on successful login
- Token contains: user ID, email, role, expiration
- Signed with HS256 algorithm
- Stored in frontend localStorage

**Token Structure:**

```
Header:  { "alg": "HS256", "typ": "JWT" }
Payload: { "id": "...", "email": "...", "role": "...", "iat": ..., "exp": ... }
Secret:  (stored in Kubernetes secret)
```

**Expiration:**

- Default: 7 days (configurable)
- Refreshable via /api/users/verify endpoint

#### **Role-Based Access Control (RBAC)**

**User Roles Defined:**

| Role           | Description          | Permissions                                           |
| -------------- | -------------------- | ----------------------------------------------------- |
| **admin**      | System administrator | All operations, user management                       |
| **manager**    | Inventory manager    | Create/update medicines, set thresholds, view reports |
| **pharmacist** | Pharmacy staff       | Stock-in/out, view inventory, fulfill orders          |
| **customer**   | End user             | Browse medicines, view profile                        |

**Protected Endpoints Example:**

```javascript
// Catalog Service Routes
router.post("/", protect, authorize("manager", "admin"), createMedicine);
router.delete("/:id", protect, authorize("admin"), deleteMedicine);

// Inventory Service Routes
router.post(
  "/stock-in",
  protect,
  authorize("pharmacist", "manager", "admin"),
  stockIn,
);
router.put(
  "/:id/threshold",
  protect,
  authorize("manager", "admin"),
  updateThreshold,
);

// User Service Routes
router.get("/profile", authMiddleware, getProfile);
```

**Authorization Middleware:**

```javascript
const authorize =
  (...allowedRoles) =>
  (req, res, next) => {
    if (!req.user) return res.status(401).json({ error: "Unauthorized" });
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: "Forbidden" });
    }
    next();
  };
```

---

### 5.2 Network Security

#### **Network Policies**

**Policy Examples (Per Microservice):**

```yaml
# Allow Catalog Service to MongoDB
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-catalog-to-mongo
spec:
  podSelector:
    matchLabels:
      app: mongo-catalog
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - catalog-service
                  - catalog-seed
```

**Implemented Policies:**

- ✅ catalog-service ↔ mongo-catalog
- ✅ user-service ↔ mongo-user
- ✅ inventory-service ↔ mongo-inventory
- ✅ frontend → all services (via gateway)

**Deny All Default:**

- By default, pods cannot communicate across namespaces
- Must explicitly allow via NetworkPolicy

#### **Container Security Context**

**Pod Security Context:**

```yaml
securityContext:
  runAsNonRoot: true # Container must run as non-root
  runAsUser: 1001 # Run as UID 1001 (non-root user)
  runAsGroup: 1001 # Run with GID 1001
  fsGroup: 1001 # Set filesystem group (MongoDB)
```

**Container Security Context:**

```yaml
securityContext:
  allowPrivilegeEscalation: false # Prevent privilege escalation
  runAsNonRoot: true # Enforce non-root
  runAsUser: 1001
  runAsGroup: 1001
  readOnlyRootFilesystem: false # (Optional) read-only filesystem
  capabilities:
    drop:
      - ALL # Drop all Linux capabilities
```

**MongoDB Specific:**

```yaml
securityContext:
  fsGroup: 999 # MongoDB default user: mongodb (UID 999)
  runAsUser: 999 # Run as mongod user
  runAsGroup: 999
  runAsNonRoot: true
```

---

### 5.3 Secret Management

#### **Kubernetes Secrets (Development)**

**Standard Kubernetes Secrets:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: rxpulse-secrets
type: Opaque
data:
  MONGO_ROOT_USERNAME: bW9uZ29hZG1pbg== # base64 encoded
  MONGO_ROOT_PASSWORD: cGFzc3dvcmQxMjM=
  MONGO_USER_URI: bW9uZ29kYjovL3...
  MONGO_CATALOG_URI: bW9uZ29kYjovL3...
  MONGO_INVENTORY_URI: bW9uZ29kYjovL3...
  JWT_SECRET: aGlnaGx5LXNlY3VyZS1zZWNyZXQ=
```

**Referenced in Pods:**

```yaml
containers:
  - name: user-service
    envFrom:
      - secretRef:
          name: rxpulse-secrets
```

#### **SealedSecrets (Production)**

**Production Secret Encryption:**

```yaml
# environments/prod/sealed-secret.yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: rxpulse-secrets
  namespace: rxpulse-prod
spec:
  encryptedData:
    MONGO_ROOT_USERNAME: AgB... (encrypted with cluster public key)
    MONGO_ROOT_PASSWORD: AgB...
    # ... all secrets encrypted
  template:
    metadata:
      name: rxpulse-secrets
      namespace: rxpulse-prod
    type: Opaque
```

**Unsealing Process:**

- SealedSecrets controller runs in cluster
- Has private key to unseal encrypted secrets
- Automatically creates regular Secret from SealedSecret
- Pods reference unsealed Secret transparently

---

### 5.4 Image Security

#### **Image Scanning**

**Trivy Scanning (in Pipeline):**

```yaml
- name: Run Trivy container image scan
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: docker.io/rxpulsehub/rxpulse-frontend:${{ inputs.image-tag }}
    format: "sarif"
    output: "trivy-results.sarif"
```

**CVE Thresholds:**

- Blocks build if CRITICAL or HIGH vulnerabilities found
- Configurable via `--severity-threshold` flag
- Reports pushed to GitHub Security tab

#### **Base Image Security**

**Production Base Images:**

| Service          | Base Image        | Reason                                    |
| ---------------- | ----------------- | ----------------------------------------- |
| Backend Services | node:18-alpine    | Minimal, non-root user: node (UID 1000)   |
| Frontend         | nginx:1.27-alpine | Minimal, runs as nginx (UID 101)          |
| MongoDB          | mongo:6.0         | Official image, runs as mongodb (UID 999) |

**Alpine Advantage:**

- ~5MB base size vs 300MB+ for standard
- Minimal attack surface
- Regular security updates

#### **Multi-Stage Dockerfile Pattern**

**Backend Example:**

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY src/ ./src/

# Stage 2: Production
FROM node:18-alpine AS production
RUN addgroup -S rxpulse && adduser -S rxpulse -G rxpulse
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/src ./src
RUN chown -R rxpulse:rxpulse /app
USER rxpulse
EXPOSE 3002
HEALTHCHECK --interval=30s ...
CMD ["node", "src/app.js"]
```

**Benefits:**

- Final image contains only production dependencies
- Eliminates dev tools (npm, git, etc.)
- Reduces image size and attack surface
- Final user: rxpulse (non-root)

---

### 5.5 TLS/mTLS

**Current State:** Not implemented in the architecture

**Recommended for Production:**

- Kubernetes Ingress TLS termination
- Service-to-service mTLS via Istio/Linkerd
- Certificate management with cert-manager

---

### 5.6 Code Quality & Security Gates

#### **SonarQube Quality Gates**

**Configured Checks:**

- Code coverage threshold: (configurable)
- Bugs: Must be 0
- Security hotspots: Manual review required
- Duplications: < 5%
- SQALE rating: A (best)

**Project Keys:**

- rxpulse-frontend
- rxpulse-user
- rxpulse-catalog
- rxpulse-inventory

**Failure Behavior:**

- Quality gate failure blocks merge to main
- Pipeline stops, email notification sent
- Developer must fix issues before retry

#### **Snyk Vulnerability Scanning**

**Configuration:**

```
--severity-threshold=high
```

**Scans:**

- package.json dependencies
- Direct and transitive vulnerabilities
- License compliance (if enabled)

**Output:**

- Snyk report JSON
- HTML report generated and stored
- Results linked in GitHub

**Fail Criteria:**

- HIGH severity or above: Block push

---

### 5.7 OWASP Top 10 Mitigations

| OWASP                          | Mitigation                              |
| ------------------------------ | --------------------------------------- |
| A1 - Broken Access Control     | JWT + RBAC + authMiddleware             |
| A2 - Cryptographic Failures    | bcryptjs for passwords, JWT secrets     |
| A3 - Injection                 | Mongoose ORM (no raw SQL)               |
| A4 - Insecure Design           | Network policies, security context      |
| A5 - Security Misconfiguration | SealedSecrets in prod, no default creds |
| A6 - Vulnerable Components     | Snyk scanning, Trivy image scan         |
| A7 - Authentication Failures   | JWT tokens, secure storage              |
| A8 - Data Integrity Failures   | HTTPS (via gateway), integrity checks   |
| A9 - Logging & Monitoring      | Container logs, health probes           |
| A10 - SSRF                     | Kubernetes network isolation            |

---

## 6. MONITORING & OBSERVABILITY

### 6.1 Health Checks

**All services expose `/health` endpoint:**

```javascript
app.get("/health", (req, res) => {
  res.status(200).json({
    status: "OK",
    service: "catalog-service",
    timestamp: Date.now(),
  });
});
```

**Kubernetes Probes:**

- **Liveness:** Restart container if unhealthy for 30s
- **Readiness:** Remove from service if unhealthy for 15s

### 6.2 Logging

**Application Logs:**

- stdout/stderr captured by container runtime
- Accessible via: `kubectl logs {pod-name} -n rxpulse-dev`
- Includes: request logs, errors, service startup messages

**Example:**

```
[catalog-service] Running on port 3002
[inventory-service] Error connecting to MongoDB
```

### 6.3 Metrics

**Currently Not Implemented:**

- No Prometheus metrics scraping configured
- No custom Grafana dashboards
- Only basic Kubernetes resource metrics available

**Recommended for Enhancement:**

- Prometheus for metrics collection
- Grafana for visualization
- Custom business metrics (API latency, request count, error rates)
- Pod resource utilization dashboards

### 6.4 Alerting

**CI/CD Pipeline Alerts:**

- Email notifications on pipeline failure
- GitHub Actions workflow status checks
- Detailed failure summaries sent to team email

**Recommended Kubernetes Alerts:**

- Pod restart frequency
- CrashLoopBackOff detection
- PVC utilization > 80%
- Node resource exhaustion

---

## 7. REPOSITORY STRUCTURE

### 7.1 Complete Directory Tree

```
RxPulse_org/
│
├── rxpulse-catalogservice/
│   ├── Dockerfile                  — Multi-stage build for catalog service
│   ├── package.json               — npm dependencies
│   ├── package-lock.json
│   ├── .env.example               — Environment template
│   │
│   └── src/
│       ├── app.js                 — Express app setup
│       ├── seed.js                — Initialize catalog with sample data
│       │
│       ├── config/
│       │   └── db.js              — MongoDB connection
│       │
│       ├── controllers/
│       │   ├── medicineController.js   — Medicine CRUD + search
│       │   └── categoryController.js   — Category CRUD
│       │
│       ├── middleware/
│       │   └── authMiddleware.js      — JWT verification + RBAC
│       │
│       ├── models/
│       │   ├── Medicine.js        — Medicine schema
│       │   └── Category.js        — Category schema
│       │
│       └── routes/
│           ├── medicineRoutes.js  — Medicine endpoints
│           └── categoryRoutes.js  — Category endpoints
│
├── rxpulse-inventoryservice/
│   ├── Dockerfile
│   ├── package.json
│   ├── .env.example
│   │
│   └── src/
│       ├── app.js                 — Express app setup
│       ├── seed.js                — Initialize inventory with sample data
│       │
│       ├── config/
│       │   └── db.js              — MongoDB connection
│       │
│       ├── controllers/
│       │   ├── stockController.js     — Stock CRUD, stock-in/out
│       │   ├── movementController.js  — Movement tracking
│       │   └── alertController.js     — Alert management + reports
│       │
│       ├── middleware/
│       │   └── authMiddleware.js      — JWT verification + RBAC
│       │
│       ├── models/
│       │   ├── Stock.js           — Stock schema
│       │   ├── Movement.js        — Movement schema
│       │   └── Alert.js           — Alert schema
│       │
│       └── routes/
│           ├── stockRoutes.js     — Stock endpoints
│           ├── movementRoutes.js  — Movement endpoints
│           └── alertRoutes.js     — Alert endpoints
│
├── rxpulse-userservice/
│   ├── Dockerfile
│   ├── package.json
│   ├── .env.example
│   │
│   └── src/
│       ├── app.js                 — Express app setup
│       ├── seed.js                — Initialize users with sample data
│       │
│       ├── config/
│       │   ├── db.js              — MongoDB connection
│       │   └── constants.js       — User roles, defaults
│       │
│       ├── controllers/
│       │   └── userController.js  — Auth, registration, profile CRUD
│       │
│       ├── middleware/
│       │   └── authMiddleware.js  — JWT verification
│       │
│       ├── models/
│       │   └── User.js            — User schema
│       │
│       └── routes/
│           └── userRoutes.js      — User endpoints
│
├── rxpulse-frontend/
│   ├── Dockerfile                  — Multi-stage: Node builder + Nginx server
│   ├── nginx.conf                  — Nginx configuration
│   ├── package.json               — npm dependencies
│   ├── package-lock.json
│   ├── index.html                 — HTML entry point
│   ├── vite.config.js             — Vite build configuration
│   ├── tailwind.config.js         — Tailwind CSS configuration
│   ├── postcss.config.js          — PostCSS configuration
│   │
│   └── src/
│       ├── main.jsx               — React entry point
│       ├── App.jsx                — Root component
│       ├── index.css              — Global styles
│       │
│       ├── api/
│       │   ├── authApi.js         — Auth endpoints wrapper
│       │   ├── catalogApi.js      — Catalog endpoints wrapper
│       │   ├── inventoryApi.js    — Inventory endpoints wrapper
│       │   └── userApi.js         — User endpoints wrapper
│       │
│       ├── context/
│       │   ├── AuthContext.jsx    — Global auth state
│       │   └── CartContext.jsx    — Global cart state
│       │
│       ├── components/
│       │   ├── admin/             — Admin UI components
│       │   ├── charts/            — Recharts visualization components
│       │   ├── common/            — Shared components (navbar, buttons, etc.)
│       │   ├── inventory/         — Inventory management components
│       │   └── medicines/         — Medicine catalog components
│       │
│       └── pages/
│           ├── Cart.jsx           — Shopping cart page
│           ├── Home.jsx           — Homepage
│           ├── Login.jsx          — Login page
│           ├── Register.jsx       — Registration page
│           ├── Shop.jsx           — Medicine shop page
│           ├── MedicineDetail.jsx — Individual medicine detail view
│           │
│           ├── admin/             — Admin pages
│           │   ├── Dashboard.jsx
│           │   ├── UserManagement.jsx
│           │   └── SystemSettings.jsx
│           │
│           ├── alerts/            — Alert notification pages
│           ├── auth/              — Authentication pages
│           ├── dashboard/         — User dashboard pages
│           ├── inventory/         — Inventory management pages
│           ├── medicines/         — Medicine management pages
│           ├── movements/         — Stock movement pages
│           └── reports/           — Reporting pages
│
├── rxpulse-helmcharts/
│   ├── README.md                   — Helm charts documentation
│   │
│   ├── charts/                     — Microservice Helm charts
│   │   ├── catalog/
│   │   │   ├── Chart.yaml         — Chart metadata
│   │   │   ├── values.yaml        — Default values
│   │   │   └── templates/
│   │   │       ├── deployment.yaml
│   │   │       ├── service.yaml
│   │   │       ├── networkpolicy.yaml
│   │   │       └── seed-job.yaml
│   │   │
│   │   ├── user/
│   │   │   ├── Chart.yaml
│   │   │   ├── values.yaml
│   │   │   └── templates/
│   │   │       ├── deployment.yaml
│   │   │       ├── service.yaml
│   │   │       ├── networkpolicy.yaml
│   │   │       └── seed-job.yaml
│   │   │
│   │   ├── inventory/
│   │   │   ├── Chart.yaml
│   │   │   ├── values.yaml
│   │   │   └── templates/
│   │   │       ├── deployment.yaml
│   │   │       ├── service.yaml
│   │   │       ├── networkpolicy.yaml
│   │   │       └── seed-job.yaml
│   │   │
│   │   └── frontend/
│   │       ├── Chart.yaml
│   │       ├── values.yaml
│   │       └── templates/
│   │           ├── rollout.yaml            — Argo Rollout blue-green
│   │           ├── hpa.yaml                — HPA autoscaler
│   │           └── service.yaml            — Service (active + preview)
│   │
│   ├── infra/                      — Infrastructure layer
│   │   ├── base/
│   │   │   ├── Chart.yaml
│   │   │   ├── values.yaml
│   │   │   └── templates/
│   │   │       ├── namespace.yaml
│   │   │       ├── configmap.yaml
│   │   │       └── serviceaccount.yaml
│   │   │
│   │   ├── gateway/                — KGateway ingress
│   │   │   ├── Chart.yaml
│   │   │   ├── values.yaml
│   │   │   └── templates/
│   │   │       ├── gateway.yaml    — KGateway resource
│   │   │       └── httproute.yaml  — HTTPRoute rules
│   │   │
│   │   └── mongodb/                — MongoDB StatefulSets
│   │       ├── Chart.yaml
│   │       ├── values.yaml
│   │       └── templates/
│   │           ├── statefulsets.yaml      — Mongo instances + services
│   │           └── storageclass.yaml      — PVC + StorageClass
│   │
│   ├── environments/
│   │   ├── dev/
│   │   │   ├── namespace.yaml
│   │   │   ├── sealed-secret.yaml         — Secrets (development)
│   │   │   └── values.yaml                — Dev-specific values
│   │   │
│   │   └── prod/
│   │       ├── namespace.yaml
│   │       ├── sealed-secret.yaml         — Secrets (encrypted for prod)
│   │       └── values.yaml                — Prod-specific values
│   │
│   └── argocd/
│       ├── project.yaml             — ArgoCD AppProject definition
│       │
│       └── applications/
│           ├── dev/
│           │   └── app.yaml         — ArgoCD Application for dev
│           │
│           └── prod/
│               └── app.yaml         — ArgoCD Application for prod
│
└── rxpulse-ci-workflows/
    ├── README.md                   — CI/CD documentation
    │
    └── .github/
        └── workflows/
            ├── code-scan.yaml       — SonarQube + Snyk scanning
            ├── build-image.yaml     — Docker build + Trivy scan
            ├── push-image.yaml      — Push to Docker Hub
            ├── update-helm.yaml     — Update Helm values
            └── notify.yaml          — Email notifications
```

---

### 7.2 Key Configuration Files Locations

| File               | Purpose                | Location                                                         |
| ------------------ | ---------------------- | ---------------------------------------------------------------- |
| Helm Charts        | K8s resource templates | rxpulse-helmcharts/charts/{service}/                             |
| Docker Builds      | Container definitions  | {service}/Dockerfile                                             |
| CI/CD Workflows    | GitHub Actions         | rxpulse-ci-workflows/.github/workflows/                          |
| Environment Values | Dev/Prod config        | rxpulse-helmcharts/environments/{env}/values.yaml                |
| ArgoCD Apps        | GitOps deployment      | rxpulse-helmcharts/argocd/applications/{env}/app.yaml            |
| Nginx Config       | Frontend proxy         | rxpulse-frontend/nginx.conf                                      |
| Network Policies   | Security policies      | rxpulse-helmcharts/charts/{service}/templates/networkpolicy.yaml |
| MongoDB Setup      | Database config        | rxpulse-helmcharts/infra/mongodb/templates/statefulsets.yaml     |
| Gateway Rules      | Ingress routing        | rxpulse-helmcharts/infra/gateway/templates/httproute.yaml        |

---

## 8. ENVIRONMENT DETAILS

### 8.1 Development Environment

| Property             | Value                                          |
| -------------------- | ---------------------------------------------- |
| **Namespace**        | rxpulse-dev                                    |
| **Helm Branch**      | develop                                        |
| **Auto-Deploy**      | Yes (automatic sync on develop branch changes) |
| **Replicas**         | Catalog: 1, User: 1, Inventory: 1, Frontend: 2 |
| **HPA Frontend**     | Min: 2, Max: 5                                 |
| **Database Storage** | 5Gi per MongoDB                                |
| **Blue-Green**       | Enabled with manual promotion                  |
| **Image Tags**       | develop-{sha} format                           |

**Access:**

- ArgoCD: Monitor in dev UI
- kubectl: `kubectl -n rxpulse-dev get pods`
- Logs: `kubectl -n rxpulse-dev logs {pod}`

### 8.2 Production Environment

| Property             | Value                                                 |
| -------------------- | ----------------------------------------------------- |
| **Namespace**        | rxpulse-prod                                          |
| **Helm Branch**      | main                                                  |
| **Auto-Deploy**      | Yes (automatic sync on main branch changes)           |
| **Replicas**         | Catalog: 1, User: 1, Inventory: 1, Frontend: 2        |
| **HPA Frontend**     | Min: 2, Max: 10                                       |
| **Database Storage** | 20Gi per MongoDB                                      |
| **Blue-Green**       | Enabled with manual promotion (required for frontend) |
| **Image Tags**       | Semantic versioning (v1.2.3) + latest                 |
| **SealedSecrets**    | Yes (encrypted at rest)                               |

**Access:**

- ArgoCD: Monitor in prod UI (requires ops role)
- kubectl: `kubectl -n rxpulse-prod get pods`
- Logs: `kubectl -n rxpulse-prod logs {pod}`

### 8.3 Cloud Provider & Infrastructure

| Property                | Value                                 |
| ----------------------- | ------------------------------------- |
| **Deployment Model**    | On-premises or local Kubernetes       |
| **Cloud Provider**      | None (self-hosted)                    |
| **Infrastructure Type** | Kubernetes cluster (single)           |
| **Storage Backend**     | NFS (nfs-client provisioner)          |
| **Container Runtime**   | Docker (assumed, Kubernetes-agnostic) |
| **Networking**          | Kubernetes CNI (assumed)              |

### 8.4 Cluster Information

- **Single Cluster Deployment** — all environments in same cluster (different namespaces)
- **Multi-Environment Strategy** — namespace isolation instead of multi-cluster
- **Network Isolation** — NetworkPolicies + Kubernetes network segmentation
- **No Multi-Cluster** — failover/disaster recovery not configured

---

## 9. DEPLOYMENT GUIDE

### 9.1 Prerequisites

```bash
# Install required tools
- kubectl (1.24+)
- Helm (3.x)
- Docker (for building images)
- Git (for cloning repositories)
- ArgoCD CLI (optional, for manual operations)
```

### 9.2 Initial Setup

#### **Step 1: Create Namespaces**

```bash
# Development
kubectl create namespace rxpulse-dev
kubectl label namespace rxpulse-dev app.kubernetes.io/part-of=rxpulse

# Production
kubectl create namespace rxpulse-prod
kubectl label namespace rxpulse-prod app.kubernetes.io/part-of=rxpulse
```

#### **Step 2: Create Kubernetes Secrets**

```bash
# Development
kubectl create secret generic rxpulse-secrets \
  --from-literal=MONGO_ROOT_USERNAME=mongoadmin \
  --from-literal=MONGO_ROOT_PASSWORD=password123 \
  --from-literal=MONGO_USER_URI=mongodb://mongoadmin:password123@mongo-user:27017/userdb \
  --from-literal=MONGO_CATALOG_URI=mongodb://mongoadmin:password123@mongo-catalog:27017/catalogdb \
  --from-literal=MONGO_INVENTORY_URI=mongodb://mongoadmin:password123@mongo-inventory:27017/inventorydb \
  --from-literal=JWT_SECRET=your-secret-key-here \
  -n rxpulse-dev
```

#### **Step 3: Deploy Infrastructure**

```bash
# Deploy base infrastructure (namespaces, configmaps)
helm install rxpulse-base ./charts/base \
  -n rxpulse-dev \
  -f environments/dev/values.yaml

# Deploy MongoDB StatefulSets
helm install rxpulse-mongodb ./infra/mongodb \
  -n rxpulse-dev \
  -f environments/dev/values.yaml

# Deploy Gateway
helm install rxpulse-gateway ./infra/gateway \
  -n rxpulse-dev \
  -f environments/dev/values.yaml
```

#### **Step 4: Deploy Services via ArgoCD**

```bash
# ArgoCD watches develop branch and automatically syncs
# Push changes to develop branch and ArgoCD syncs services
kubectl apply -f argocd/project.yaml
kubectl apply -f argocd/applications/dev/app.yaml
```

### 9.3 Verify Deployment

```bash
# Check pods are running
kubectl get pods -n rxpulse-dev
kubectl get pods -n rxpulse-prod

# Check services
kubectl get svc -n rxpulse-dev

# Check Rollouts
kubectl argo rollouts list -n rxpulse-dev

# Check HPA
kubectl get hpa -n rxpulse-dev

# View logs
kubectl logs -f deployment/catalog-service -n rxpulse-dev
```

### 9.4 Blue-Green Promotion (Frontend)

```bash
# View current promotion status
kubectl argo rollouts get rollout frontend -n rxpulse-dev

# Promote green to blue (after testing)
kubectl argo rollouts promote frontend -n rxpulse-dev

# Rollback to previous version
kubectl argo rollouts abort rollout frontend -n rxpulse-dev
```

---

## 10. ARCHITECTURE DIAGRAMS

### 10.1 Application Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Frontend Layer                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  React SPA (Nginx) - Port 8080                             │ │
│  │  ├─ Authentication Pages                                  │ │
│  │  ├─ Medicine Browser                                      │ │
│  │  ├─ Inventory Dashboard                                   │ │
│  │  ├─ Admin Panel                                           │ │
│  │  └─ Reports & Charts                                      │ │
│  └──────────────────────────────┬────────────────────────────┘ │
└──────────────────────────────────┼───────────────────────────────┘
                                   │ HTTP REST API
                                   │ (JSON)
                 ┌─────────────────┼─────────────────┐
                 │                 │                 │
      ┌──────────▼────────┐  ┌─────▼────────────┐  ┌───────▼──────────┐
      │ User Service      │  │ Catalog Service  │  │ Inventory Service │
      │ (Port 3001)       │  │ (Port 3002)      │  │ (Port 3003)       │
      ├───────────────────┤  ├──────────────────┤  ├──────────────────┤
      │ • Register        │  │ • Get Medicines  │  │ • Get Stocks     │
      │ • Login           │  │ • Search         │  │ • Record Movement│
      │ • Verify Token    │  │ • Create Item    │  │ • Set Alerts     │
      │ • Get/Update      │  │ • Delete Item    │  │ • Generate       │
      │   Profile         │  │ • Categories     │  │   Reports        │
      └──────────┬────────┘  └────────┬─────────┘  └────────┬─────────┘
                 │ MongoDB             │ MongoDB            │ MongoDB
                 │                     │                    │
      ┌──────────▼────────┐  ┌─────▼────────────┐  ┌───────▼──────────┐
      │ MongoDB User      │  │ MongoDB Catalog  │  │ MongoDB Inventory │
      │ (StatefulSet)     │  │ (StatefulSet)    │  │ (StatefulSet)     │
      │ Database: userdb  │  │ Database:        │  │ Database:         │
      │ PVC: 5-20Gi       │  │ catalogdb        │  │ inventorydb       │
      │                   │  │ PVC: 5-20Gi      │  │ PVC: 5-20Gi       │
      └───────────────────┘  └──────────────────┘  └───────────────────┘
```

---

### 10.2 Kubernetes Architecture Diagram

```
┌────────────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                              │
│                                                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │              Namespace: rxpulse-dev / rxpulse-prod          │ │
│  │                                                             │ │
│  │  ┌─────────────────────────────────────────────────────┐   │ │
│  │  │          KGateway (Ingress)                         │   │ │
│  │  │  Listeners: HTTP:80                                │   │ │
│  │  │  HTTPRoutes:                                        │   │ │
│  │  │  • /api/users → user-service:3001                 │   │ │
│  │  │  • /api/catalog → catalog-service:3002            │   │ │
│  │  │  • /api/inventory → inventory-service:3003        │   │ │
│  │  │  • / → frontend-active:8080                        │   │ │
│  │  └─────────────┬───────────────────────────────────────┘   │ │
│  │                │                                           │ │
│  │    ┌───────────┼────────────┬─────────────┬──────────────┐ │ │
│  │    │           │            │             │              │ │ │
│  │ ┌──▼─────┐ ┌──▼──────┐ ┌──▼────────┐ ┌──▼────────────┐ │ │ │
│  │ │ User   │ │Catalog  │ │Inventory  │ │Frontend      │ │ │ │
│  │ │Service │ │Service  │ │Service    │ │(Argo Rollout)│ │ │ │
│  │ │Deploy. │ │Deploy.  │ │Deploy.    │ │              │ │ │ │
│  │ │(1 pod) │ │(1 pod)  │ │(1 pod)    │ │Blue → Green  │ │ │ │
│  │ │        │ │         │ │           │ │HPA: 2-5/10   │ │ │ │
│  │ └──┬─────┘ └──┬──────┘ └──┬────────┘ └──┬───────────┘ │ │ │
│  │    │          │           │            │              │ │ │
│  │ ┌──▼─────┐ ┌──▼──────┐ ┌──▼────────┐   │              │ │ │
│  │ │MongoDB │ │MongoDB  │ │MongoDB    │   │              │ │ │
│  │ │User    │ │Catalog  │ │Inventory  │   │              │ │ │
│  │ │StatefulSet      │ │Stateful    │   │              │ │ │
│  │ │(1 pod)│ │(1 pod)  │ │Set (1 pod)│   │              │ │ │
│  │ │       │ │         │ │           │   │              │ │ │
│  │ │PVCs   │ │PVCs     │ │PVCs       │   │              │ │ │
│  │ └───────┘ └─────────┘ └───────────┘   │              │ │ │
│  │                                         │              │ │ │
│  │  NetworkPolicies:                      │              │ │ │
│  │  ├─ allow-user-to-mongo               │              │ │ │
│  │  ├─ allow-catalog-to-mongo            │              │ │ │
│  │  └─ allow-inventory-to-mongo          │              │ │ │
│  │                                         │              │ │ │
│  │  ConfigMaps:                           │              │ │ │
│  │  └─ rxpulse-config (env vars)         │              │ │ │
│  │                                         │              │ │ │
│  │  Secrets:                              │              │ │ │
│  │  └─ rxpulse-secrets (DB creds, JWT)   │              │ │ │
│  └─────────────────────────────────────────────────────────┘ │ │
│                                                              │ │
│  ┌─────────────────────────────────────────────────────────┐ │ │
│  │          ArgoCD (GitOps Controller)                     │ │ │
│  │  Watches: rxpulse-helmcharts repo                      │ │ │
│  │  Sync: Automatic (develop & main branches)            │ │ │
│  │  Strategy: Blue-Green (Frontend), Rolling (Services)  │ │ │
│  └─────────────────────────────────────────────────────────┘ │ │
│                                                              │ │
└──────────────────────────────────────────────────────────────┘ │
                                                                 │
  Storage Backend: NFS                                         │
  Storage Class: nfs-client                                    │
  PV/PVC Provisioning: Dynamic                                 │
```

---

### 10.3 CI/CD Pipeline Flow Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                      Developer Workflow                          │
│                                                                  │
│  Branch Type: Feature / Develop ──→ Main (Production)           │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       │ Push / Create PR
                       ▼
    ┌──────────────────────────────────────────────────────────┐
    │         GitHub Actions Trigger                           │
    │  (branch: develop, main, feature/*, release/*, hotfix/*)│
    └─────────────────┬────────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
    ┌───────┐    ┌──────────┐  ┌────────────┐
    │ build-│    │code-scan │  │push-image  │
    │image  │    │.yaml     │  │(after push)│
    │.yaml  │    │          │  │            │
    └───┬───┘    └────┬─────┘  └─────┬──────┘
        │             │              │
        │ Parallel    │              │
        ▼             ▼              ▼
    ┌──────────────────────────────────────┐
    │  1. code-scan.yaml                   │
    │  ├─ SonarQube Analysis               │
    │  │  ├─ npm ci                        │
    │  │  ├─ sonarqube-scan-action         │
    │  │  └─ quality-gate check (5 min)   │
    │  │  └─ FAIL: Stop pipeline          │
    │  │                                   │
    │  └─ Snyk Vulnerability Scan          │
    │     ├─ npm install                   │
    │     ├─ snyk test --severity-threshold=high
    │     ├─ snyk-to-html conversion       │
    │     └─ FAIL (HIGH CVE): Stop        │
    └──────────────────┬───────────────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
        ▼                             ▼
    ┌──────────────────┐  ┌──────────────────────┐
    │ 2. build-image   │  │ 3. push-image        │
    │    .yaml         │  │    .yaml             │
    ├──────────────────┤  ├──────────────────────┤
    │ generate-tag:    │  │ Download artifact    │
    │ ├─ main: semver  │  │ Load Docker image    │
    │ │  (v1.2.3)      │  │ Login Docker Hub     │
    │ └─ other:        │  │                      │
    │    branch-sha    │  │ If main branch:      │
    │                  │  │ ├─ Push: {user}/... │
    │ build-image:     │  │ │  :v1.2.3          │
    │ ├─ Multi-stage   │  │ ├─ Push: {user}/... │
    │ │  Dockerfile    │  │ │  :latest          │
    │ ├─ artifact       │  │ ├─ Create git tag   │
    │ │  storage        │  │ │  v1.2.3           │
    │ └─ Docker load    │  │ └─ Push tag origin  │
    │                  │  │                      │
    │ trivy scan:      │  │ If other branch:     │
    │ ├─ Image scan    │  │ └─ Push: {user}/... │
    │ ├─ CVE check     │  │    :branch-sha      │
    │ └─ FAIL: Report  │  │                      │
    │                  │  │ FAIL: Stop pipeline  │
    └────────┬─────────┘  └──────────┬───────────┘
             │                       │
             └───────────┬───────────┘
                        │ Success
                        ▼
    ┌──────────────────────────────────────────────────┐
    │  4. update-helm.yaml                             │
    │                                                  │
    │  Checkout: rxpulse-helmcharts repo               │
    │  ├─ If main: environments/prod/values.yaml       │
    │  └─ If develop: environments/dev/values.yaml     │
    │                                                  │
    │  Update:                                         │
    │  global.services.{service}.tag = {new-tag}       │
    │                                                  │
    │  Commit: "chore({service}): update image tag"    │
    │  Push: to target branch (main/develop)           │
    │                                                  │
    │  FAIL: Stop (values file not found)              │
    └─────────────┬──────────────────────────────────┘
                  │ Helm values committed
                  ▼
    ┌──────────────────────────────────────────────────┐
    │  5. ArgoCD Detects Change                        │
    │                                                  │
    │  Watches: rxpulse-helmcharts repo                │
    │  ├─ Watches: main (prod) & develop (dev)         │
    │  ├─ Polls every 3 minutes                        │
    │  └─ Detects: new image tag in values.yaml        │
    │                                                  │
    │  Syncs to Cluster:                               │
    │  ├─ If dev: syncs to rxpulse-dev namespace       │
    │  └─ If prod: syncs to rxpulse-prod namespace     │
    │                                                  │
    │  Frontend Deployment (Argo Rollout):             │
    │  ├─ Blue-Green Strategy                          │
    │  ├─ Deploy green version                         │
    │  ├─ Wait for manual promotion (prod)             │
    │  └─ Promote green → blue (traffic switch)        │
    │                                                  │
    │  Other Services (Standard Deployment):           │
    │  └─ Rolling update (kill old, start new)         │
    └─────────────┬──────────────────────────────────┘
                  │ Deployment complete
                  ▼
    ┌──────────────────────────────────────────────────┐
    │  6. notify.yaml                                  │
    │                                                  │
    │  Determine Status:                               │
    │  ├─ Check all stage results                      │
    │  ├─ SUCCESS: All stages passed                   │
    │  └─ FAILED: At least one failure                 │
    │                                                  │
    │  Send Email:                                     │
    │  ├─ To: team email (NOTIFY_TO_EMAIL)             │
    │  ├─ Subject: [RxPulse CI/CD] {STATUS} ...        │
    │  └─ Body: Detailed results table                 │
    │                                                  │
    │  Failure Summary:                                │
    │  └─ Lists failed jobs with reasons               │
    └──────────────────────────────────────────────────┘
                        │
                        ▼
           ┌────────────────────────┐
           │    Deployment Live     │
           │  in Dev or Production  │
           └────────────────────────┘
```

---

## Summary

RxPulse is a **production-grade pharmacy management platform** built with:

- **4 Microservices** (User, Catalog, Inventory) + 1 **React Frontend**
- **Kubernetes-native** deployment with **Helm** charts
- **GitOps** via **ArgoCD** for automated deployments
- **Blue-Green deployments** for frontend using **Argo Rollouts**
- **Comprehensive CI/CD** via **GitHub Actions** with SonarQube, Snyk, Trivy
- **Security-first** design: JWT auth, RBAC, NetworkPolicies, SealedSecrets, non-root containers
- **Scalable architecture** with HPA, multi-stage Docker builds, efficient resource management
- **Production-ready** monitoring, logging, and health checks

This architecture demonstrates modern DevOps practices, cloud-native design, and enterprise-grade security implementations suitable for a capstone project.

---

**Document Version:** 1.0  
**Last Updated:** April 2026  
