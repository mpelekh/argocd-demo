# ArgoCD Demo

This repository contains a demo setup for ArgoCD using Kind clusters.

## Setup Steps

### ArgoCD Cluster Setup

1. Create a Kind cluster for ArgoCD:
   ```bash
   kind create cluster --config kind-clusters/argocd-cluster.yaml
   ```

2. Set up kubectl alias for the ArgoCD cluster:
   ```bash
   alias kk="kubectl --context kind-argocd-cluster"
   ```

3. Create the ArgoCD namespace:
   ```bash
   kk create namespace argocd
   ```

4. Install ArgoCD using the official manifests:
   ```bash
   kk apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/refs/tags/v2.14.11/manifests/install.yaml
   ```

5. Patch ArgoCD config for faster reconciliation:
   ```bash
   kk -n argocd patch configmap argocd-cm --type merge -p '{"data":{"timeout.reconciliation":"180s"}}'
   ```

6. Verify ArgoCD installation and set up port forwarding:
   ```bash
   # Check all ArgoCD resources
   kk -n argocd get all

   # Check ArgoCD services
   kk -n argocd get svc

   # Set up port forwarding to access ArgoCD UI
   kk -n argocd port-forward service/argocd-server 8443:443
   ```

7. Retrieve the initial admin password:
   ```bash
   kk -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
   ```

### Test Cluster Setup

1. Create a Kind cluster for testing:
   ```bash
   kind create cluster --config kind-clusters/test-cluster.yaml
   ```

2. Set up kubectl alias for the test cluster:
   ```bash
   alias kt="kubectl --context kind-test-cluster"
   ```

3. Add the test cluster to ArgoCD declaratively:
   ```bash
   kk apply -f argocd/test-cluster-secret.yaml
   ```

4. Add the Kwok cluster to ArgoCD declaratively:
   ```bash
   kk apply -f argocd/kwok-cluster-secret.yaml
   ```

### Kwok Cluster Setup (Demo)

1. Create a Kwok cluster for demo purposes:
   ```bash
   kwokctl create cluster --runtime=kind
   ```

2. Set up kubectl alias for the Kwok cluster:
   ```bash
   alias kw="kubectl --context kwok-cluster"
   ```

3. Scale the number of nodes in the Kwok cluster:
   ```bash
   kwokctl scale node --replicas=100
   ```

4. Add the Kwok cluster to ArgoCD declaratively:
   ```bash
   kk apply -f argocd/kwok-cluster-secret.yaml
   ```

## Deploying Applications

After adding the test and Kwok clusters to ArgoCD, you can deploy applications to them using ArgoCD. This can be done either through:

1. The ArgoCD UI
2. ArgoCD CLI commands
3. Declarative configuration files

Example of deploying the demo application:
```bash
kk apply -f argocd/application/demo-app.yaml
```

The application will be deployed to the kwok cluster, and ArgoCD will continuously monitor and reconcile the desired state.
