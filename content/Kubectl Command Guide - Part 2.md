
## Service Management

### Basic Service Operations
```bash
# Create service
kubectl expose deployment DEPLOYMENT_NAME \
    --port=80 \
    --target-port=8080 \
    --type=ClusterIP

# Create service with advanced options
kubectl expose deployment DEPLOYMENT_NAME \
    --port=443 \
    --target-port=8443 \
    --type=LoadBalancer \
    --name=SERVICE_NAME \
    --external-ip=EXTERNAL_IP \
    --load-balancer-ip=LOAD_BALANCER_IP

# Get service information
kubectl get svc
kubectl get svc -o wide
kubectl get endpoints SERVICE_NAME
```

### Advanced Service Configuration
```bash
# Create service with multiple ports
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  selector:
    app: my-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
  - name: metrics
    port: 9090
    targetPort: 9090
EOF

# Create ExternalName service
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  type: ExternalName
  externalName: api.external-service.com
EOF

# Create headless service
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None
  selector:
    app: stateful-app
  ports:
  - port: 80
    targetPort: 80
EOF
```

## Configuration Management

### ConfigMaps
```bash
# Create ConfigMap
kubectl create configmap CONFIG_NAME \
    --from-file=config.properties \
    --from-literal=key1=value1 \
    --from-literal=key2=value2

# Create ConfigMap from directory
kubectl create configmap CONFIG_NAME \
    --from-file=config-dir/

# Use ConfigMap in pod
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: app
    image: nginx
    envFrom:
    - configMapRef:
        name: CONFIG_NAME
    volumeMounts:
    - name: config-volume
      mountPath: /config
  volumes:
  - name: config-volume
    configMap:
      name: CONFIG_NAME
EOF
```

### Secrets
```bash
# Create secret
kubectl create secret generic SECRET_NAME \
    --from-file=ssh-privatekey=~/.ssh/id_rsa \
    --from-literal=password=mysecretpassword

# Create TLS secret
kubectl create secret tls TLS_SECRET \
    --cert=path/to/tls.cert \
    --key=path/to/tls.key

# Use secret in pod
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: SECRET_NAME
          key: password
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: SECRET_NAME
EOF
```

## Resource Management

### Resource Quotas
```bash
# Create ResourceQuota
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 4Gi
    limits.cpu: "8"
    limits.memory: 8Gi
    pods: "10"
EOF

# Create LimitRange
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-range
spec:
  limits:
  - type: Container
    default:
      cpu: 200m
      memory: 512Mi
    defaultRequest:
      cpu: 100m
      memory: 256Mi
    max:
      cpu: 1
      memory: 1Gi
    min:
      cpu: 50m
      memory: 128Mi
EOF
```

### Network Policies
```bash
# Create NetworkPolicy
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
EOF
```

## Advanced Operations

### RBAC Configuration
```bash
# Create ServiceAccount
kubectl create serviceaccount SA_NAME

# Create Role
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
EOF

# Create RoleBinding
kubectl create rolebinding NAME \
    --clusterrole=view \
    --serviceaccount=NAMESPACE:SA_NAME \
    --namespace=NAMESPACE

# Create ClusterRole
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
EOF
```

### Custom Resource Definitions
```bash
# Create CRD
cat <<EOF | kubectl apply -f -
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: websites.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                url:
                  type: string
                replicas:
                  type: integer
  scope: Namespaced
  names:
    plural: websites
    singular: website
    kind: Website
    shortNames:
    - web
EOF
```

