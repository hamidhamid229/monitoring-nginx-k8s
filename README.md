# Monitoring Setup with Helm

Hello!
This repository is designed to deploy a monitoring stack using Helm.

Step 1: Install Helm Charts

First, add the Prometheus community Helm repository:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts/
```
#Step 2: Deploy with Custom Values

Next, install the monitoring stack using your custom values.yaml file:
```
helm install monitoring prometheus-community/kube-prometheus-stack -f values.yaml
```
# Step 3: Apply Required Manifests

Before running the above commands, make sure to apply the following Kubernetes manifests:
```
README.md  
nginx-exporter-pod.yaml  
nginx-exporter-service.yaml  
nginx-pod.yaml  
nginx-service.yaml  
values.yaml
```




