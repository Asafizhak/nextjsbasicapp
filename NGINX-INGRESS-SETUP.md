# NGINX Ingress Setup Guide

## Changes Made for NGINX Ingress

### 1. Service Updated
- **Changed**: `type: LoadBalancer` → `type: ClusterIP`
- **File**: `k8s/service.yaml`
- **Reason**: NGINX Ingress Controller handles external traffic, service only needs internal cluster access

### 2. Ingress Resource Created
- **File**: `k8s/ingress.yaml`
- **Host**: `nextjsbasicapp.local` (temporary for testing)
- **Path**: `/` (root path)

### 3. Workflow Updated
- **Added**: Ingress verification step
- **File**: `.github/workflows/deploy-to-aks.yml`

## How to Access Your Application

### Option 1: Local Testing with /etc/hosts
1. Get NGINX Ingress Controller external IP:
   ```bash
   kubectl get svc -n ingress-nginx
   ```

2. Add to your local `/etc/hosts` file (Linux/Mac) or `C:\Windows\System32\drivers\etc\hosts` (Windows):
   ```
   <EXTERNAL-IP> nextjsbasicapp.local
   ```

3. Access: `http://nextjsbasicapp.local`

### Option 2: Direct IP Access (for testing)
1. Get the external IP:
   ```bash
   kubectl get svc -n ingress-nginx
   ```

2. Access directly: `http://<EXTERNAL-IP>`

## DNS Setup for Production

### When Ready for Real Domain:

1. **Update ingress.yaml**:
   ```yaml
   spec:
     rules:
     - host: your-real-domain.com  # Change this
   ```

2. **Configure DNS**:
   - Point your domain A record to NGINX Ingress external IP
   - Or use CNAME to point to Azure's FQDN

3. **Add SSL (Optional)**:
   ```yaml
   metadata:
     annotations:
       cert-manager.io/cluster-issuer: "letsencrypt-prod"
   spec:
     tls:
     - hosts:
       - your-real-domain.com
       secretName: nextjsbasicapp-tls
   ```

## Architecture After Changes

```
Internet → NGINX Ingress Controller → nextjsbasicapp Service (ClusterIP) → Pods
```

## Verification Commands

```bash
# Check ingress status
kubectl get ingress -n nextjsbasicapp

# Check ingress details
kubectl describe ingress nextjsbasicapp-ingress -n nextjsbasicapp

# Check NGINX controller
kubectl get svc -n ingress-nginx

# Check pods
kubectl get pods -n nextjsbasicapp
```

## Troubleshooting

### If application is not accessible:
1. Verify NGINX Ingress Controller is running
2. Check ingress resource is created
3. Verify service endpoints
4. Check DNS resolution (if using domain)

### Common Issues:
- **404 errors**: Check ingress path configuration
- **Connection refused**: Verify NGINX controller external IP
- **DNS issues**: Check /etc/hosts or DNS records