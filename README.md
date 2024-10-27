# Argo CD Installation Guide

This guide explains how to install Argo CD on a Kubernetes cluster and expose it using an Ingress.

## Prerequisites
- A Kubernetes cluster (with `kubectl` access)
- An Ingress controller installed in the cluster (e.g., NGINX Ingress Controller)
- A DNS name pointing to your Ingress controller's external IP (e.g., `argocd.example.com`)

## Installation Steps

### Install Argo CD
Install Argo CD in a new namespace called `argocd`:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
### Expose Argo CD using Ingress

Create an Ingress Resource
Create a file named argocd-ingress.yaml with the following content:
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  rules:
    - host: argocd.example.com  # Replace with your domain name
      http:
        paths:
          - path: /
            pathType: Prefix
            backend
              service:
                name: argocd-server
                port:
                  number: 443
  tls:
    - hosts:
        - argocd.example.com  # Replace with your domain name
      secretName: argocd-tls
```
### Create a TLS Secret (Optional)
If you have an SSL certificate and key, create a Kubernetes secret:
```bash
kubectl create secret tls argocd-tls --cert=path/to/tls.crt --key=path/to/tls.key -n argocd
```
Alternatively, you can use cert-manager to automatically generate a TLS certificate.
### 3. Login to Argo CD
Get the initial admin password:
```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```
### Access Argo CD:
Navigate to https://argocd.example.com in your web browser.
Login with the username admin and the password retrieved above.
Change the admin password (recommended):
```bash
argocd login <ARGOCD_SERVER> --username admin --password <INITIAL_PASSWORD>
argocd account update-password
```

