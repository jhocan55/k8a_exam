# Troubleshooting Guide

This guide provides solutions to common issues encountered during the deployment and operation of the FastAPI application using the Kubernetes Helm chart.

---

## 1. Helm Release Issues

### Problem: Helm Release Fails to Install
- **Cause**: Missing or incorrect values in `values.yaml`.
- **Solution**:
  1. Check the Helm release status:
     ```bash
     helm status fastapi -n <namespace>
     ```
  2. Review the logs for errors:
     ```bash
     helm get notes fastapi -n <namespace>
     ```

### Problem: Helm Upgrade Fails
- **Cause**: Conflicting resource configurations.
- **Solution**:
  1. Roll back to the previous release:
     ```bash
     helm rollback fastapi <revision> -n <namespace>
     ```
  2. Fix the conflicting configurations in `values.yaml` and retry.

---

## 2. Pod Issues

### Problem: Pods Are Not Starting
- **Cause**: Image pull errors or insufficient resources.
- **Solution**:
  1. Check pod status:
     ```bash
     kubectl get pods -n <namespace>
     ```
  2. Inspect pod logs:
     ```bash
     kubectl logs <pod-name> -n <namespace>
     ```
  3. Verify the image repository and tag in `values.yaml`:
     ```yaml
     image:
       repository: yourdockerhub/fastapi-app
       tag: latest
     ```

### Problem: CrashLoopBackOff
- **Cause**: Application misconfiguration or missing environment variables.
- **Solution**:
  1. Check the pod logs for errors:
     ```bash
     kubectl logs <pod-name> -n <namespace>
     ```
  2. Verify the environment variables in `values.yaml`:
     ```yaml
     env:
       - name: DATABASE_URL
         value: "postgresql://user:password@db:5432/mydatabase"
     ```

---

## 3. Service Issues

### Problem: Service Not Accessible
- **Cause**: Incorrect service type or port configuration.
- **Solution**:
  1. Verify the service configuration:
     ```bash
     kubectl get svc -n <namespace>
     ```
  2. Check the `service` section in `values.yaml`:
     ```yaml
     service:
       type: ClusterIP
       port: 80
     ```

### Problem: LoadBalancer Service Not Getting External IP
- **Cause**: Cloud provider integration issues.
- **Solution**:
  1. Check the service status:
     ```bash
     kubectl describe svc <service-name> -n <namespace>
     ```
  2. Ensure your Kubernetes cluster is configured with a cloud provider that supports LoadBalancer services.

---

## 4. Ingress Issues

### Problem: Ingress Not Working
- **Cause**: Missing or misconfigured Ingress controller.
- **Solution**:
  1. Verify the Ingress controller is installed:
     ```bash
     kubectl get pods -n ingress-nginx
     ```
  2. Check the Ingress resource:
     ```bash
     kubectl get ingress -n <namespace>
     ```
  3. Verify the `ingress` section in `values.yaml`:
     ```yaml
     ingress:
       enabled: true
       host: app1.mydomain.com
     ```

### Problem: TLS Not Working
- **Cause**: Missing or invalid TLS secret.
- **Solution**:
  1. Verify the TLS secret exists:
     ```bash
     kubectl get secret fastapi-cert -n <namespace>
     ```
  2. Check the certificate and key files used to create the secret:
     ```bash
     kubectl describe secret fastapi-cert -n <namespace>
     ```

---

## 5. Exposing Issues

### Problem: Cloudflare Tunnel Not Working
- **Cause**: Incorrect tunnel configuration.
- **Solution**:
  1. Verify the `cloudflared` client is authenticated:
     ```bash
     cloudflared tunnel list
     ```
  2. Check the tunnel URL and Kubernetes service name.

### Problem: Ngrok Tunnel Not Accessible
- **Cause**: Incorrect service URL or network issues.
- **Solution**:
  1. Verify the Ngrok tunnel is running:
     ```bash
     ngrok http http://<service-name>.<namespace>.svc.cluster.local:80
     ```
  2. Ensure the Kubernetes service is reachable internally.

---

## 6. General Debugging Tips

### Check Kubernetes Events
- Use the following command to view cluster events:
  ```bash
  kubectl get events -n <namespace>
  ```

### Describe Resources
- Use `kubectl describe` to get detailed information about any resource:
  ```bash
  kubectl describe <resource-type> <resource-name> -n <namespace>
  ```

### Inspect Logs
- Check logs for specific pods or controllers:
  ```bash
  kubectl logs <pod-name> -n <namespace>
  ```

---

Refer to the [Exposing the Application Guide](./exposing.md) and [Helm Chart Templates Guide](./templates.md) for additional context.
