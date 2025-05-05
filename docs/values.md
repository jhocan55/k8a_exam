# Values Configuration Guide

The `values.yaml` file is the main configuration file for the Helm chart. It allows you to customize the deployment of your FastAPI application. Below is a detailed explanation of its key sections, along with examples and advanced configurations.

---

## 1. `replicaCount`

This value defines the number of replicas (pods) for the FastAPI application. Increasing the replica count improves availability and scalability.

```yaml
replicaCount: 3
```

### Example:
- **Development**: Use `replicaCount: 1` to save resources.
- **Production**: Use `replicaCount: 3` or more for high availability.

---

## 2. `image`

This section specifies the Docker image and tag for the FastAPI application.

```yaml
image:
  repository: yourdockerhub/fastapi-app
  tag: latest
```

### Fields:
- `repository`: The Docker repository where your FastAPI image is stored.
- `tag`: The version of the image to use.

### Example:
```yaml
image:
  repository: mydockerhub/fastapi-app
  tag: v1.0.0
```

### Notes:
- Ensure the image is publicly accessible or provide Kubernetes with credentials to access private repositories.

---

## 3. `service`

This section configures the Kubernetes Service that exposes the application internally within the cluster.

```yaml
service:
  type: ClusterIP
  port: 80
```

### Fields:
- `type`: Defines the service type (`ClusterIP`, `NodePort`, or `LoadBalancer`).
- `port`: The port on which the service is exposed.

### Example:
- **ClusterIP**: Use for internal communication within the cluster.
- **NodePort**: Use for exposing the application on a specific port of the cluster nodes.
- **LoadBalancer**: Use for cloud environments to expose the application externally.

```yaml
service:
  type: LoadBalancer
  port: 8080
```

---

## 4. `ingress`

This section configures the Ingress resource for external access to the application.

```yaml
ingress:
  enabled: true
  host: app1.mydomain.com
```

### Fields:
- `enabled`: Enables or disables the Ingress resource.
- `host`: The domain name for the application.

### Example:
```yaml
ingress:
  enabled: true
  host: fastapi.example.com
```

### Notes:
- Ensure an Ingress controller (e.g., NGINX Ingress) is installed in your cluster.
- Use a wildcard domain for dynamic environments (e.g., `*.example.com`).

---

## 5. `tls`

This section configures TLS for secure HTTPS access to the application.

```yaml
tls:
  enabled: true
  secretName: fastapi-cert
```

### Fields:
- `enabled`: Enables or disables TLS.
- `secretName`: The name of the Kubernetes secret containing the TLS certificate and key.

### Example:
```yaml
tls:
  enabled: true
  secretName: my-tls-secret
```

### Notes:
- Create the TLS secret before deploying the Helm chart:
  ```bash
  kubectl create secret tls fastapi-cert --cert=path/to/cert.crt --key=path/to/key.key -n <namespace>
  ```

---

## Advanced Configurations

### 1. Environment Variables

You can pass environment variables to the FastAPI application by extending the Helm chart templates. Add the following to the `values.yaml` file:

```yaml
env:
  - name: DATABASE_URL
    value: "postgresql://user:password@db:5432/mydatabase"
```

### 2. Resource Limits

Define resource requests and limits for the application pods:

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1"
```

### 3. Custom Annotations

Add custom annotations to Kubernetes resources:

```yaml
annotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "80"
```

---

## Common Mistakes and Troubleshooting

### 1. Missing Docker Image
- Ensure the `repository` and `tag` fields in the `image` section are correct.
- Verify the image exists in the specified Docker registry.

### 2. Ingress Not Working
- Check if the Ingress controller is installed and running.
- Verify the domain name in the `host` field.

### 3. TLS Issues
- Ensure the TLS secret exists in the correct namespace.
- Verify the certificate and key files are valid.

---

Refer to the [Helm Chart Templates Guide](./templates.md) to see how these values are used in the templates.
