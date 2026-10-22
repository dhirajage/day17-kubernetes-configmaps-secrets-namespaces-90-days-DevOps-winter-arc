# Day 18: ConfigMaps, Secrets & Namespaces (Kubeadm on AWS EC2)

## Objective
Practice environment-specific configuration isolation, secret management, environment variable injection, mounted configuration files, and rolling updates for configuration changes on a self-managed Kubernetes cluster built with Kubeadm on AWS EC2.

## Architecture & Key Concepts
- **Namespaces:** Provide logical resource isolation within the same physical cluster, separating environments like development (`dev`) and production (`prod`).
- **ConfigMaps:** Store non-sensitive key-value application settings and file content decoupled from container images.
- **Secrets:** Store sensitive data (such as database credentials) securely, encoded in base64 by default.
- **Environment Variable Injection:** Exposes configuration maps and secrets directly into container runtimes via `env` and `valueFrom`.
- **Mounted Configuration Files:** Mounts ConfigMap data as files into container volume paths (e.g., `/usr/share/nginx/html`), allowing automatic updates for files without container restarts.
- **Update Behavior Differences:** ConfigMap volumes mounted as directories automatically sync changes after a short delay, whereas environment variables require a pod/deployment rollout restart to take effect.

## Isolated Namespaces Creation (Task A)
- Created dedicated `dev` and `prod` namespaces to establish strict environment boundaries.
- Verified active status and existence using `kubectl get namespaces`.

## Environment-Specific ConfigMaps (Task B)
- Created environment-tailored ConfigMaps (`site`) containing `APP_ENV` values and embedded `index.html` web page templates (`Hello from DEV` vs `Hello from PROD`) for both namespaces.

## Secure Lab Secrets Creation (Task C)
- Generated generic `db-secret` objects in both `dev` and `prod` namespaces using dummy literal values (`dev-pass` and `prod-pass`) to handle credentials securely.

## Unified Workload Deployment (Task D)
- Applied `deployment.yaml` across both namespaces, running `nginx:1.27` with environment variables injected from ConfigMaps and Secrets, alongside volume mounts for the web root.

## Port-Forwarding & Environment Verification (Task E)
- Forwarded ports (`8091` for dev, `8082` for prod) using separate terminal sessions.
- Tested web pages via `curl` and verified namespace-specific environment variables (`APP_ENV` and `DB_PASSWORD`) using `kubectl exec`.

## Configuration Update & Rollout Restart (Task F)
- Edited the development ConfigMap directly (`kubectl edit configmap site -n dev`) to observe mounted file synchronization.
- Performed a rollout restart (`kubectl rollout restart deployment/web -n dev`) to successfully update container environment variables.

## Interview / Reflection Answers

### 1. What is the difference between a ConfigMap and a Secret?
A ConfigMap is used for non-sensitive configuration data stored in plain text, while a Secret is intended for sensitive data (like passwords, keys, or tokens) and is base64-encoded by default.

### 2. Why must referenced ConfigMaps and Secrets exist in the Pod's namespace?
Kubernetes configuration objects are namespace-scoped. Pods cannot reference or import ConfigMaps or Secrets that belong to a different namespace.

### 3. What is the difference between environment variables and mounted files?
Environment variables are injected into container memory at startup and do not dynamically update when the ConfigMap changes. Mounted files in volumes automatically update after a short sync delay without requiring a container restart (unless mounted using `subPath`).

### 4. Why does changing a ConfigMap not update an existing container's environment automatically?
Environment variables are set during process initialization when the container starts up. The container's environment context cannot be dynamically re-injected by Kubernetes without restarting the process.

### 5. Why is base64 not sufficient to protect a Secret?
Base64 is merely an encoding algorithm, not encryption. Anyone with read access to the Secret manifest can easily decode it back into plain text. Real security requires encryption at rest in etcd and strict RBAC controls.

### 6. What can cause `CreateContainerConfigError`?
This error usually occurs when a Pod references a ConfigMap or Secret that does not exist in the same namespace, or when key names inside the manifest do not match the key names defined in the configuration object.

### 7. How do namespaces help separate dev and prod resources?
Namespaces provide virtual boundary isolation on a shared physical cluster, allowing resource name overlap, independent access controls (RBAC), and specific resource quotas across different deployment environments.

## Repository Structure
```text
day18-configmaps-secrets-namespaces/
├── README.md
├── configmap-dev.yaml
├── configmap-prod.yaml
├── deployment.yaml
└── evidence/
    ├── 1.png
    ├── 2.png
    ├── 3.png
    ├── 4.png
    ├── 5.png
    ├── 6.png
    └── 7.png