# FastAPI Kubernetes Deployment Guide (From Scratch)

This guide explains how to deploy a FastAPI application using Kubernetes (`k3s`) and Docker on a fresh system. It assumes that Docker and `k3s` are already installed.

---

## Prerequisites

1. **Docker Installed**: Ensure Docker is installed and running. Verify with:
   ```bash
   docker --version
   ```

2. **k3s Installed**: Ensure `k3s` is installed and running. Verify with:
   ```bash
   kubectl get nodes
   ```

3. **Helm Installed**: Install Helm, the Kubernetes package manager:
   ```bash
   curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
   ```
   Verify the installation:
   ```bash
   helm version
   ```

---

## Step 1: Clone the Repository

Clone the repository containing the Helm chart:
```bash
git clone <repository-url>
cd /home/alex/Documents/Kubernetes2/charts/fastapi
```

---

## Step 2: Build and Push the FastAPI Docker Image

1. **Build the Docker Image**:
   Ensure your FastAPI application code is in the same directory as the `Dockerfile`. Build the image:
   ```bash
   docker build -t yourdockerhub/fastapi-app:latest .
   ```

2. **Push the Image to Docker Hub**:
   Log in to Docker Hub and push the image:
   ```bash
   docker login
   docker push yourdockerhub/fastapi-app:latest
   ```

---

## Step 3: Deploy PostgreSQL (Optional)

If your FastAPI application requires a PostgreSQL database, deploy it in the Kubernetes cluster:

1. **Install PostgreSQL Using Helm**:
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm install postgresql bitnami/postgresql --namespace fastapi --create-namespace
   ```

2. **Retrieve PostgreSQL Credentials**:
   ```bash
   export POSTGRES_PASSWORD=$(kubectl get secret --namespace fastapi postgresql -o jsonpath="{.data.postgresql-password}" | base64 --decode)
   echo "PostgreSQL password: $POSTGRES_PASSWORD"
   ```

3. **Update `values.yaml`**:
   Add the PostgreSQL connection string to the `env` section:
   ```yaml
   env:
     - name: DATABASE_URL
       value: "postgresql://postgres:$POSTGRES_PASSWORD@postgresql.fastapi.svc.cluster.local:5432/mydatabase"
   ```

---

## Step 4: Customize `values.yaml`

Edit the `values.yaml` file to match your requirements. Below are the key sections to configure:

- **replicaCount**: Number of replicas (pods) for the application.
- **image.repository**: Your Docker image repository.
- **image.tag**: The tag of your Docker image.
- **ingress.host**: The domain name for your application.
- **tls.secretName**: The name of the TLS secret for HTTPS.

Example:
```yaml
replicaCount: 2
image:
  repository: yourdockerhub/fastapi-app
  tag: latest
service:
  type: ClusterIP
  port: 80
ingress:
  enabled: true
  host: fastapi.local
tls:
  enabled: true
  secretName: fastapi-cert
```

---

## Step 5: Deploy the FastAPI Application

1. **Install the Helm Chart**:
   Deploy the FastAPI application using Helm:
   ```bash
   helm install fastapi ./ --namespace fastapi --create-namespace
   ```

2. **Verify the Deployment**:
   Check the status of the deployment:
   ```bash
   kubectl get all -n fastapi
   ```

---

## Step 6: Expose the Application

### Option 1: Using Ingress (Recommended for Production)
1. Ensure the `k3s` Ingress controller is enabled (it is enabled by default in `k3s`).
2. Access the application using the domain specified in `values.yaml` (e.g., `http://fastapi.local`).

### Option 2: Using NodePort (For Development)
1. Update the `service.type` in `values.yaml` to `NodePort`:
   ```yaml
   service:
     type: NodePort
   ```
2. Redeploy the Helm chart:
   ```bash
   helm upgrade fastapi ./ --namespace fastapi
   ```
3. Access the application using the node's IP and the assigned NodePort.

### Option 3: Using Ngrok (For Quick Testing)
1. Install Ngrok:
   ```bash
   wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-stable-linux-amd64.zip
   unzip ngrok-stable-linux-amd64.zip
   sudo mv ngrok /usr/local/bin/
   ```
2. Start a tunnel to the service:
   ```bash
   ngrok http http://<service-name>.fastapi.svc.cluster.local:80
   ```
3. Use the generated Ngrok URL to access the application.

---

## Step 7: Verify the Application

Access your application using the method you chose in the previous step:
- **Ingress**: `http://fastapi.local`
- **NodePort**: `http://<node-ip>:<node-port>`
- **Ngrok**: `https://<ngrok-id>.ngrok.io`

---

## Troubleshooting

If you encounter any issues, refer to the [Troubleshooting Guide](./docs/troubleshooting.md) or use the following commands to debug:

- Check the status of the Helm release:
  ```bash
  helm status fastapi -n fastapi
  ```
- Inspect pod logs:
  ```bash
  kubectl logs -l app.kubernetes.io/name=fastapi -n fastapi
  ```
- View Kubernetes events:
  ```bash
  kubectl get events -n fastapi
  ```

---

## Conclusion

This guide provides a step-by-step process to deploy and expose your FastAPI application using `k3s` and Docker. By following these steps, you can easily set up a scalable and production-ready deployment. For advanced configurations, refer to the additional guides in the `docs/` folder.