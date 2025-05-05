# Exposing the Application Guide

This guide explains how to expose your FastAPI application to the internet. You can use tools like Cloudflare Tunnel or Ngrok to make your application accessible externally.

---

## Why Expose the Application?

By default, Kubernetes services like `ClusterIP` are only accessible within the cluster. To make your application accessible to external users, you need to expose it using one of the following methods:
- **Ingress**: For production environments with a domain name.
- **Cloudflare Tunnel**: For secure tunneling without exposing your cluster's IP.
- **Ngrok**: For quick and temporary access during development.

---

## Option 1: Using Cloudflare Tunnel

Cloudflare Tunnel provides a secure way to expose your application without opening any ports on your cluster.

### Steps:
1. **Install Cloudflare Tunnel Client**:
   Download and install the `cloudflared` client:
   ```bash
   wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
   sudo mv cloudflared-linux-amd64 /usr/local/bin/cloudflared
   sudo chmod +x /usr/local/bin/cloudflared
   ```

2. **Authenticate with Cloudflare**:
   Log in to your Cloudflare account:
   ```bash
   cloudflared login
   ```

3. **Create a Tunnel**:
   Create a tunnel to your Kubernetes service:
   ```bash
   cloudflared tunnel --url http://<service-name>.<namespace>.svc.cluster.local:80
   ```

4. **Update DNS Records**:
   In your Cloudflare dashboard, create a CNAME record pointing to the tunnel.

### Example:
If your service is named `fastapi-service` in the `default` namespace:
```bash
cloudflared tunnel --url http://fastapi-service.default.svc.cluster.local:80
```

---

## Option 2: Using Ngrok

Ngrok is a lightweight tool for exposing local services to the internet. It is ideal for development and testing.

### Steps:
1. **Install Ngrok**:
   Download and install Ngrok:
   ```bash
   wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-stable-linux-amd64.zip
   unzip ngrok-stable-linux-amd64.zip
   sudo mv ngrok /usr/local/bin/
   ```

2. **Start a Tunnel**:
   Start a tunnel to your Kubernetes service:
   ```bash
   ngrok http http://<service-name>.<namespace>.svc.cluster.local:80
   ```

3. **Access the Application**:
   Ngrok will provide a public URL (e.g., `https://<ngrok-id>.ngrok.io`) that you can use to access your application.

### Example:
If your service is named `fastapi-service` in the `default` namespace:
```bash
ngrok http http://fastapi-service.default.svc.cluster.local:80
```

---

## Option 3: Using Kubernetes Ingress

For production environments, use Kubernetes Ingress to expose your application with a domain name.

### Prerequisites:
- An Ingress controller (e.g., NGINX Ingress) installed in your cluster.
- A domain name pointing to your cluster's external IP.

### Steps:
1. **Enable Ingress in `values.yaml`**:
   ```yaml
   ingress:
     enabled: true
     host: app1.mydomain.com
   tls:
     enabled: true
     secretName: fastapi-cert
   ```

2. **Deploy the Helm Chart**:
   Apply the changes by redeploying the Helm chart:
   ```bash
   helm upgrade fastapi ./ --namespace <namespace>
   ```

3. **Verify Ingress**:
   Check the Ingress resource:
   ```bash
   kubectl get ingress -n <namespace>
   ```

4. **Access the Application**:
   Use the domain name (e.g., `https://app1.mydomain.com`) to access your application.

---

## Comparison of Methods

| Method              | Use Case                     | Pros                              | Cons                              |
|---------------------|------------------------------|-----------------------------------|-----------------------------------|
| **Cloudflare Tunnel** | Secure, production-ready    | No open ports, secure tunneling  | Requires Cloudflare account       |
| **Ngrok**            | Development and testing     | Quick setup, free tier available | Temporary, not production-ready   |
| **Ingress**          | Production environments     | Full control, domain support     | Requires Ingress controller setup |

---

## Troubleshooting

### 1. Cloudflare Tunnel Not Working
- Ensure the `cloudflared` client is authenticated with your Cloudflare account.
- Verify the Kubernetes service name and namespace.

### 2. Ngrok Tunnel Not Accessible
- Check if the Kubernetes service is running and reachable.
- Ensure Ngrok is configured to point to the correct service URL.

### 3. Ingress Issues
- Verify that the Ingress controller is installed and running.
- Check the DNS configuration for your domain.

---

Refer to the [Troubleshooting Guide](./troubleshooting.md) for additional help.
