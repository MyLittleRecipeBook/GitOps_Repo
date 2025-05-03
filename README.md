# My Little Recipe Book: A Distributed Kubernetes-based Recipe Management System

My Little Recipe Book is a comprehensive recipe management application deployed on Kubernetes that provides recipe storage, OCR-based recipe scanning, and AI-powered recipe recommendations. The system integrates multiple services including frontend, backend APIs, OCR processing, and database management with high availability and scalability features.

The application is built as a microservices architecture running on Amazon EKS, featuring separate frontend and backend services, OCR processing capabilities, and integration with multiple authentication providers (Kakao, Naver, Google). It includes automatic scaling, load balancing, and persistent storage management for reliable recipe data storage and retrieval.

## Repository Structure
```
.
├── Infrastructure Configuration
│   ├── 01-Namespace.yaml           # Defines frontend, backend, and database namespaces
│   ├── 02-Configmap.yaml           # Application configuration for frontend and backend services
│   ├── 03-Secret.yaml             # Sensitive configuration data for authentication and databases
│   ├── 03-StorageClass.yaml       # EBS storage configuration for persistent volumes
│   └── 04-PV-PVC.yaml            # Persistent volume claims for database storage
├── Service Deployments
│   ├── be_fap_deployment.yaml     # FastAPI backend service deployment
│   ├── be_nod_deployment.yaml     # Node.js backend service deployment
│   ├── be_ocr_deployment.yaml     # OCR processing service deployment
│   ├── db_statefulset.yaml       # Database StatefulSet configuration
│   └── fe_deployment.yaml         # Frontend service deployment
├── Networking
│   ├── 08-Ingress.yaml           # External access configuration
│   ├── 09-Service.yaml           # Internal service definitions
│   └── 14-Netpol.yaml            # Network policies for service communication
└── Resource Management
    ├── 12-HPA.yaml               # Horizontal Pod Autoscaling configuration
    ├── 13-LimitRage.yaml         # Container resource constraints
    └── 15-ResourceQuota.yaml     # Namespace resource quotas
```

## Usage Instructions
### Prerequisites
- AWS Account with EKS access
- kubectl CLI tool (v1.30.0 or later)
- AWS CLI configured with appropriate credentials
- Docker registry access (ECR)
- Helm (optional, for package management)

### Installation

1. Configure AWS CLI and authenticate:
```bash
aws configure
aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${AWS_EKS_REGION}
```

2. Create required namespaces:
```bash
kubectl apply -f 01-Namespace.yaml
```

3. Configure storage and secrets:
```bash
kubectl apply -f 03-StorageClass.yaml
kubectl apply -f 03-Secret.yaml
kubectl apply -f 04-PV-PVC.yaml
```

4. Deploy core services:
```bash
kubectl apply -f be_nod_deployment.yaml
kubectl apply -f be_fap_deployment.yaml
kubectl apply -f be_ocr_deployment.yaml
kubectl apply -f fe_deployment.yaml
kubectl apply -f db_statefulset.yaml
```

### Quick Start
1. Verify all services are running:
```bash
kubectl get pods -A
```

2. Access the application:
```bash
kubectl get ingress -n mlr-prd-fe-ns
```
Use the provided URL to access the frontend application.

### More Detailed Examples
1. Scaling services:
```bash
kubectl scale deployment mlr-prd-be-dpl-nod -n mlr-prd-be-ns --replicas=5
```

2. Checking logs:
```bash
kubectl logs -f deployment/mlr-prd-be-dpl-nod -n mlr-prd-be-ns
```

### Troubleshooting
1. Pod Startup Issues
- Check pod status:
```bash
kubectl describe pod <pod-name> -n <namespace>
```
- Verify ConfigMaps and Secrets are properly mounted:
```bash
kubectl get configmaps -n <namespace>
kubectl get secrets -n <namespace>
```

2. Network Connectivity
- Verify network policies:
```bash
kubectl get networkpolicies -n <namespace>
```
- Test service connectivity:
```bash
kubectl run temp-pod --rm -i --tty --image=busybox -- /bin/sh
```

## Data Flow
The application processes recipe data through multiple services, from user input through OCR processing to storage and retrieval.

```ascii
User -> Frontend -> [Auth Service] -> Backend API
                                 -> OCR Service
                                 -> FastAPI Service -> Redis Cache
                                                  -> MySQL Database
```

Key component interactions:
1. Frontend communicates with backend services through REST APIs
2. Authentication flows through OAuth2 providers (Kakao, Naver, Google)
3. OCR service processes image uploads for recipe extraction
4. FastAPI service handles recipe recommendations and search
5. Redis provides caching for frequently accessed data
6. MySQL stores persistent recipe and user data
7. All services communicate over internal Kubernetes networking

## Infrastructure

![Infrastructure diagram](./docs/infra.svg)
The application runs on AWS EKS with the following key components:

Lambda Functions:
- Authentication handlers for OAuth providers
- Image processing triggers

EKS Resources:
- Multiple node groups for different workloads
- Autoscaling groups for dynamic capacity
- ALB Ingress Controller for traffic management

Storage:
- EBS volumes for persistent storage
- S3 buckets for image storage
- Redis for caching