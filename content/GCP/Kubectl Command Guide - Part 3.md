
## Troubleshooting and Debugging

### Pod Troubleshooting
```bash
# Debug running pod
kubectl debug POD_NAME -it \
    --image=busybox \
    --target=CONTAINER_NAME

# Create debugging pod
kubectl run debug-pod \
    --image=nicolaka/netshoot \
    --rm -it -- /bin/bash

# Check pod events
kubectl get events --field-selector involvedObject.name=POD_NAME

# Get pod details
kubectl get pod POD_NAME -o yaml
kubectl describe pod POD_NAME

# Check pod logs with previous instances
kubectl logs POD_NAME --previous
kubectl logs POD_NAME -c CONTAINER_NAME --previous

# Port forward for debugging
kubectl port-forward pod/POD_NAME 8080:80
```

### Cluster Diagnostics
```bash
# Check cluster health
kubectl get componentstatuses
kubectl get nodes -o custom-columns=NAME:.metadata.name,STATUS:.status.conditions[?(@.type=="Ready")].status

# Check system pods
kubectl get pods -n kube-system
kubectl top nodes
kubectl top pods --all-namespaces

# API server availability
kubectl get --raw /healthz
kubectl get --raw /metrics

# Check cluster events
kubectl get events --sort-by=.metadata.creationTimestamp

# Audit pod placement
kubectl get pods -o wide --all-namespaces | \
    grep -v Running | \
    grep -v Completed
```

### Network Diagnostics
```bash
# Test network connectivity
kubectl run test-dns \
    --image=busybox:1.28 \
    --rm -it \
    -- nslookup kubernetes.default

# Check service endpoints
kubectl get endpoints SERVICE_NAME
kubectl describe endpoints SERVICE_NAME

# Test service connectivity
kubectl run curl \
    --image=curlimages/curl \
    --rm -it \
    -- curl http://SERVICE_NAME.NAMESPACE.svc.cluster.local

# DNS debugging
kubectl run dnsutils \
    --image=gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 \
    --rm -it \
    -- bash
```

## Monitoring and Logging

### Resource Monitoring
```bash
# Monitor resource usage
kubectl top nodes --sort-by=cpu
kubectl top pods --sort-by=memory

# Get metrics for specific pod
kubectl top pod POD_NAME --containers

# Monitor pod resource usage over time
kubectl top pods --all-namespaces \
    --sort-by=cpu \
    --no-headers | \
    while read line; do
        echo "$(date '+%Y-%m-%d %H:%M:%S') $line" >> pod_metrics.log
    done
```

### Log Collection
```bash
# Collect logs from all containers
kubectl logs -l app=nginx --all-containers=true

# Export logs to file
kubectl logs POD_NAME > pod.log

# Stream logs from multiple pods
kubectl logs -f -l app=nginx --all-containers=true

# Get logs with timestamp
kubectl logs POD_NAME --timestamps=true

# Aggregate logs script
cat <<'EOF' > collect-logs.sh
#!/bin/bash
NAMESPACE=$1
LABEL_SELECTOR=$2
OUTPUT_DIR="cluster-logs-$(date +%Y%m%d-%H%M%S)"

mkdir -p "$OUTPUT_DIR"

# Collect pod logs
kubectl get pods -n "$NAMESPACE" -l "$LABEL_SELECTOR" -o name | \
while read pod; do
    pod_name=$(basename "$pod")
    mkdir -p "$OUTPUT_DIR/$pod_name"
    
    # Get pod details
    kubectl describe pod "$pod_name" -n "$NAMESPACE" > \
        "$OUTPUT_DIR/$pod_name/describe.txt"
    
    # Get container logs
    kubectl get pod "$pod_name" -n "$NAMESPACE" \
        -o jsonpath='{.spec.containers[*].name}' | \
    tr ' ' '\n' | \
    while read container; do
        kubectl logs "$pod_name" -c "$container" -n "$NAMESPACE" \
            --timestamps > \
            "$OUTPUT_DIR/$pod_name/$container.log"
    done
done

# Collect events
kubectl get events -n "$NAMESPACE" \
    --sort-by='.lastTimestamp' > \
    "$OUTPUT_DIR/events.txt"

# Create archive
tar czf "$OUTPUT_DIR.tar.gz" "$OUTPUT_DIR"
rm -rf "$OUTPUT_DIR"
EOF

chmod +x collect-logs.sh
```

## Advanced Scripting and Automation

### Deployment Automation
```bash
# Rolling update script
cat <<'EOF' > rolling-update.sh
#!/bin/bash
DEPLOYMENT=$1
NEW_IMAGE=$2
NAMESPACE=${3:-default}

# Update image
kubectl set image deployment/$DEPLOYMENT \
    *=$NEW_IMAGE -n $NAMESPACE

# Watch rollout status
kubectl rollout status deployment/$DEPLOYMENT -n $NAMESPACE

# Verify deployment
if [ $? -eq 0 ]; then
    echo "Deployment successful"
    # Run tests here
else
    echo "Deployment failed"
    kubectl rollout undo deployment/$DEPLOYMENT -n $NAMESPACE
    exit 1
fi
EOF

# Health check script
cat <<'EOF' > health-check.sh
#!/bin/bash
NAMESPACE=${1:-default}
THRESHOLD=${2:-80}

check_resources() {
    # Check CPU usage
    kubectl top pods -n $NAMESPACE | \
    awk -v threshold=$THRESHOLD 'NR>1 {
        split($3, cpu, "m")
        if (cpu[1] > threshold) {
            print $1 " CPU: " $3
        }
    }'

    # Check memory usage
    kubectl top pods -n $NAMESPACE | \
    awk -v threshold=$THRESHOLD 'NR>1 {
        split($4, mem, "Mi")
        if (mem[1] > threshold) {
            print $1 " Memory: " $4
        }
    }'
}

# Check pod status
kubectl get pods -n $NAMESPACE | \
awk 'NR>1 && $3!="Running" && $3!="Completed" {
    print "Pod " $1 " is in state " $3
}'

# Run resource checks
check_resources
EOF
```

### Backup and Restore
```bash
# Backup script
cat <<'EOF' > backup-resources.sh
#!/bin/bash
BACKUP_DIR="k8s-backup-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Backup all resources
for resource in $(kubectl api-resources --verbs=list --namespaced -o name); do
    echo "Backing up $resource"
    kubectl get "$resource" \
        --all-namespaces \
        -o yaml > \
        "$BACKUP_DIR/$resource.yaml"
done

# Backup non-namespaced resources
for resource in $(kubectl api-resources --verbs=list --namespaced=false -o name); do
    echo "Backing up $resource"
    kubectl get "$resource" \
        -o yaml > \
        "$BACKUP_DIR/$resource.yaml"
done

# Create archive
tar czf "$BACKUP_DIR.tar.gz" "$BACKUP_DIR"
rm -rf "$BACKUP_DIR"
EOF

# Restore script
cat <<'EOF' > restore-resources.sh
#!/bin/bash
BACKUP_FILE=$1

if [ ! -f "$BACKUP_FILE" ]; then
    echo "Backup file not found"
    exit 1
fi

# Extract backup
tar xzf "$BACKUP_FILE"
BACKUP_DIR=$(basename "$BACKUP_FILE" .tar.gz)

# Restore resources
for yaml in "$BACKUP_DIR"/*.yaml; do
    echo "Restoring $(basename "$yaml")"
    kubectl apply -f "$yaml"
done

rm -rf "$BACKUP_DIR"
EOF
```

## Best Practices

### Security Best Practices
```bash
# Pod security context
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
EOF

# Network policies
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF
```

### Resource Management Best Practices
```bash
# Resource requests and limits
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
EOF

# Horizontal Pod Autoscaling
cat <<EOF | kubectl apply -f -
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
EOF
```

Remember:
- Always use resource requests and limits
- Implement proper security contexts
- Use network policies
- Regular backup of critical resources
- Monitor resource usage
- Implement proper logging
- Use namespaces for isolation
- Regular security audits
- Document all configurations
- Use version control for manifests

For detailed information, consult the Kubernetes documentation and kubectl command reference (`kubectl help`).

Would you like me to cover any specific aspect in more detail?