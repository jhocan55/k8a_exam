# Helm Chart Templates Guide

The Helm chart templates define the Kubernetes resources for the FastAPI application. These templates use placeholders that are replaced with values from `values.yaml` during deployment. Below is a detailed explanation of the key templates, including support for Horizontal Pod Autoscaler (HPA).

---

## 1. Deployment Template

The Deployment template defines the pods and their configuration. It ensures that the desired number of replicas are running and manages updates to the application.

```yaml
# ...existing code...
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
      - name: fastapi
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.port }}
        env:
        {{- if .Values.env }}
        {{- toYaml .Values.env | nindent 8 }}
        {{- end }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
# ...existing code...
```

### Key Features:
- **Replicas**: Controlled by `{{ .Values.replicaCount }}` from `values.yaml`.
- **Image**: Pulled from `{{ .Values.image.repository }}` and `{{ .Values.image.tag }}`.
- **Environment Variables**: Dynamically injected if defined in `values.yaml`.
- **Resource Limits**: Configured using the `resources` section in `values.yaml`.

---

## 2. Service Template

The Service template exposes the application internally within the cluster. It defines how other services or pods can communicate with the FastAPI application.

```yaml
# ...existing code...
spec:
  type: {{ .Values.service.type }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: {{ .Values.service.port }}
  selector:
    app: {{ .Chart.Name }}
# ...existing code...
```

### Key Features:
- **Type**: Configured using `{{ .Values.service.type }}` (e.g., `ClusterIP`, `NodePort`, or `LoadBalancer`).
- **Port**: Exposed on `{{ .Values.service.port }}`.

---

## 3. Ingress Template

The Ingress template provides external access to the application. It maps HTTP/HTTPS requests to the FastAPI service.

```yaml
# ...existing code...
spec:
  rules:
  - host: {{ .Values.ingress.host }}
    http:
      paths:
      - path: /
        pathType: ImplementationSpecific
        backend:
          service:
            name: {{ .Chart.Name }}
            port:
              number: {{ .Values.service.port }}
  {{- if .Values.tls.enabled }}
  tls:
  - hosts:
    - {{ .Values.ingress.host }}
    secretName: {{ .Values.tls.secretName }}
  {{- end }}
# ...existing code...
```

### Key Features:
- **Host**: Configured using `{{ .Values.ingress.host }}`.
- **TLS**: Enabled if `{{ .Values.tls.enabled }}` is set to `true`, using the secret defined in `{{ .Values.tls.secretName }}`.

---

## 4. Horizontal Pod Autoscaler (HPA) Template

The HPA template automatically adjusts the number of pods in the Deployment based on CPU or memory utilization.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ .Chart.Name }}
  namespace: {{ .Release.Namespace }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ .Chart.Name }}
  minReplicas: {{ .Values.hpa.minReplicas }}
  maxReplicas: {{ .Values.hpa.maxReplicas }}
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: {{ .Values.hpa.cpuUtilization }}
```

### Key Features:
- **Min/Max Replicas**: Configured using `{{ .Values.hpa.minReplicas }}` and `{{ .Values.hpa.maxReplicas }}`.
- **CPU Utilization**: Target CPU utilization is set using `{{ .Values.hpa.cpuUtilization }}`.

---

## Advanced Features

### 1. Custom Annotations

Annotations can be added to Kubernetes resources for integrations with monitoring tools like Prometheus or external DNS providers.

Example:
```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "{{ .Values.service.port }}"
```

---

## Common Mistakes and Troubleshooting

### 1. Missing Placeholders
- Ensure all placeholders in the templates (e.g., `{{ .Values.image.repository }}`) are defined in `values.yaml`.

### 2. Incorrect Resource Names
- Verify that the resource names in the templates match the expected names in your cluster.

### 3. Ingress Issues
- Ensure the Ingress controller is installed and running in your cluster.
- Verify that the domain name in `{{ .Values.ingress.host }}` resolves to the cluster's external IP.

---

Refer to the [Values Configuration Guide](./values.md) for details on how to configure the placeholders used in these templates.