### Step 1: Install ArgoCD
You can install ArgoCD in your Kubernetes cluster using the following commands:

```bash
# Create the ArgoCD namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Step 2: Verify ArgoCD Installation
After the installation is complete, you can check if all pods are running:

```bash
kubectl get pods -n argocd
```

### Step 3: ArgoCD Server Ports
By default, the `argocd-server` service listens on the following ports:
- **HTTP**: 8080
- **HTTPS**: 443

### Step 4: Change ArgoCD Server Service to NodePort
To access the ArgoCD UI from outside the cluster, change the service type from `ClusterIP` to `NodePort`. You can modify the service using the following command:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort", "ports": [{"port": 443, "nodePort": 32766}]}}}'
```

### Step 5: Access ArgoCD UI
You can access the ArgoCD UI using the following URL:
```
https://<node-ip>:32766
```
Replace `<node-ip>` with the IP address of any node in your Kubernetes cluster.

### Step 6: Retrieve Initial Admin Password
To retrieve the initial admin password for ArgoCD, run the following command:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o json | jq .data.password -r | base64 -d
```

This command decodes the base64-encoded password stored in the secret.

### Step 7: Install ArgoCD CLI
To install the ArgoCD CLI, run the following commands:

```bash
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.11/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```

### Step 8: Create an ArgoCD Application
1. **Login to ArgoCD CLI**:
   First, login to ArgoCD using the CLI:

   ```bash
   argocd login <node-ip>:32766 --username admin --password <initial-admin-password>
   ```

2. **Create an Application**:
   You can create an application using the following command. Adjust the `spec.source` and `spec.destination` to match your GitHub repository and the Kubernetes cluster namespace.

   ```bash
   argocd app create my-three-tier-app \
     --repo https://github.com/joisyousef/Orange-kubernetes-Project.git \
     --path manifests/ \
     --dest-server https://kubernetes.default.svc \
     --dest-namespace default \
     --sync-policy automated
   ```

   - Replace `your-username` and `your-repo` with your GitHub username and repository name.
   - Ensure that the path in the repository contains the YAML manifests for your three-tier application.

3. **Sync the Application**:
   Sync the application to deploy it:

   ```bash
   argocd app sync my-three-tier-app
   ```

4. **Verify the Application**:
   Check the status of the application to ensure it is running:

   ```bash
   argocd app get my-three-tier-app
   ```

### Step 9: Managing Secrets in Kubernetes
To manage secrets that should not be stored in your GitHub repository, you can use Kubernetes Secrets. Here's how:

1. **Create a Secret**:
   Use the following command to create a secret from literal values:

   ```bash
   kubectl create secret generic my-secret \
     --from-literal=secret-key=my-secret-value \
     -n <your-namespace>
   ```

2. **Reference the Secret in Your YAML Manifests**:
   In your application manifests, reference the secret like this:

   ```yaml
   apiVersion: v1
   kind: Deployment
   metadata:
     name: my-app
     namespace: webapp
   spec:
     template:
       spec:
         containers:
         - name: my-container
           image: joisyousef/backend
           env:
           - name: SECRET_KEY
             valueFrom:
               secretKeyRef:
                 name: my-secret
                 key: secret-key
   ```

3. **Store Secrets Securely**:
   You can use tools like [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) or [External Secrets](https://github.com/external-secrets/external-secrets) to manage and securely store your secrets outside of your GitHub repository.

