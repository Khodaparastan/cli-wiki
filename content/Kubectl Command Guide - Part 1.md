## Table of Contents
1. [Basic Operations](#basic-operations)
2. [Pod Management](#pod-management)
3. [Deployment Management](#deployment-management)
4. [Service Management](#service-management)
5. [Configuration Management](#configuration-management)
6. [Resource Management](#resource-management)
7. [Cluster Operations](#cluster-operations)
8. [Advanced Operations](#advanced-operations)
9. [Troubleshooting](#troubleshooting)
10. [Best Practices](#best-practices)

## Basic Operations

### Context and Configuration
```bash
# View and manage contexts
kubectl config view
kubectl config get-contexts
kubectl config use-context CONTEXT_NAME
kubectl config set-context --current --namespace=NAMESPACE

# Create context with specific credentials
kubectl config set-credentials USER_NAME \
    --client-certificate=path/to/cert.crt \
    --client-key=path/to/key.key

# Set cluster details
kubectl config set-cluster CLUSTER_NAME \
    --server=https://kubernetes.example.com \
    --certificate-authority=path/to/ca.crt \
    --embed-certs=true

# Create context
kubectl config set-context CONTEXT_NAME \
    --cluster=CLUSTER_NAME \
    --user=USER_NAME \
    --namespace=NAMESPACE
```

### Basic Resource Operations
```bash
# Get resources
kubectl get pods,deployments,services -n NAMESPACE
kubectl get all -A
kubectl get pods -o wide
kubectl get pods -o json
kubectl get pods -o yaml

# Describe resources
kubectl describe pod POD_NAME
kubectl describe node NODE_NAME
kubectl describe deployment DEPLOYMENT_NAME

# Create resources
kubectl create -f manifest.yaml
kubectl apply -f manifest.yaml
kubectl apply -f directory/ -R

# Delete resources
kubectl delete -f manifest.yaml
kubectl delete pod POD_NAME --grace-period=0 --force
kubectl delete namespace NAMESPACE --timeout=5m
```

## Pod Management

### Pod Operations
```bash
# Create pod
kubectl run nginx --image=nginx:latest --port=80

# Create pod with advanced options
kubectl run custom-pod \
    --image=nginx:latest \
    --port=80 \
    --labels="app=web,env=prod" \
    --requests="cpu=100m,memory=256Mi" \
    --limits="cpu=200m,memory=512Mi" \
    --env="KEY1=VALUE1,KEY2=VALUE2" \
    --command -- /bin/sh -c "while true; do echo hello; sleep 10; done"

# Get pod details
kubectl get pod POD_NAME -o yaml
kubectl get pods --show-labels
kubectl get pods -l app=nginx
kubectl get pods --field-selector status.phase=Running

# Pod logs
kubectl logs POD_NAME
kubectl logs POD_NAME -c CONTAINER_NAME
kubectl logs -f POD_NAME
kubectl logs POD_NAME --tail=100
kubectl logs -l app=nginx --all-containers=true

# Execute commands in pod
kubectl exec -it POD_NAME -- /bin/bash
kubectl exec POD_NAME -c CONTAINER_NAME -- command
kubectl exec POD_NAME -- env
```

### Advanced Pod Configuration
```bash
# Create pod with volume
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: pvc-name
EOF

# Create pod with init container
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  initContainers:
  - name: init
    image: busybox
    command: ['sh', '-c', 'until nslookup service; do echo waiting; sleep 2; done;']
  containers:
  - name: app
    image: nginx
EOF

# Create pod with security context
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: nginx
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
EOF
```

## Deployment Management

### Deployment Operations
```bash
# Create deployment
kubectl create deployment nginx --image=nginx:latest --replicas=3

# Create deployment with advanced options
kubectl create deployment web-app \
    --image=nginx:latest \
    --replicas=3 \
    --port=80 \
    --dry-run=client -o yaml > deployment.yaml

# Scale deployment
kubectl scale deployment DEPLOYMENT_NAME --replicas=5
kubectl scale deployment DEPLOYMENT_NAME --replicas=5 --current-replicas=3

# Update deployment
kubectl set image deployment/DEPLOYMENT_NAME CONTAINER=IMAGE:TAG
kubectl rollout status deployment/DEPLOYMENT_NAME
kubectl rollout history deployment/DEPLOYMENT_NAME

# Rollback deployment
kubectl rollout undo deployment/DEPLOYMENT_NAME
kubectl rollout undo deployment/DEPLOYMENT_NAME --to-revision=2
```

### Advanced Deployment Configuration
```bash
# Deployment with rolling update strategy
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 200m
            memory: 512Mi
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 15
EOF

# Deployment with HPA
kubectl autoscale deployment DEPLOYMENT_NAME \
    --min=3 \
    --max=10 \
    --cpu-percent=80

# Create deployment with custom scheduler
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-scheduler-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: custom-app
  template:
    metadata:
      labels:
        app: custom-app
    spec:
      schedulerName: custom-scheduler
      containers:
      - name: nginx
        image: nginx:latest
EOF
```

