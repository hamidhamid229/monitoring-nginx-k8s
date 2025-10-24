# =====================================================================
# Project: Prometheus + Grafana Helm Setup for NGINX Exporter Testing
# Description:
#   This setup deploys Prometheus and Grafana using the kube-prometheus-stack
#   Helm chart. The configuration (values.yaml) is optional and intended
#   for testing an NGINX exporter inside a Kubernetes cluster.
# =====================================================================

# -----------------------------
# Step 1: Clone project files
# -----------------------------
# The repository contains:
#   - nginx-pod.yaml
#   - nginx-service.yaml
#   - nginx-exporter-pod.yaml
#   - nginx-exporter-service.yaml
#   - values.yaml
#
# Example:
#   git clone <your-repo-url>
#   cd <your-repo>
#   ls
#   # nginx-exporter-pod.yaml  nginx-exporter-service.yaml  nginx-pod.yaml  nginx-service.yaml  values.yaml

# -----------------------------
# Step 2: Install Helm chart
# -----------------------------
# Commands:
#   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
#   helm repo update
#   helm install prometheus prometheus-community/kube-prometheus-stack -f values.yaml
#
# The configuration below enables persistent storage, Grafana via NodePort,
# and adds an optional scrape job for nginx-exporter.

# -----------------------------
# Helm values.yaml configuration
# -----------------------------
prometheus:
  prometheusSpec:
    retention: 7d
    serviceMonitorSelectorNilUsesHelmValues: false
    podMonitorSelectorNilUsesHelmValues: false
    scrapeInterval: 30s
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: "nfs-client"
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 8Gi
    additionalScrapeConfigs:
      - job_name: 'nginx-exporter'
        scrape_interval: 15s
        static_configs:
          - targets:
              - 'nginx-exporter-service.default.svc.cluster.local:9113'

grafana:
  enabled: true
  adminUser: admin
  adminPassword: admin
  service:
    type: NodePort  # NodePort (auto-assigned port)
  initChownData:
    enabled: false
  podSecurityContext:
    fsGroup: 472
    runAsUser: 0
  persistence:
    enabled: true
    type: pvc
    storageClassName: "nfs-client"
    accessModes:
      - ReadWriteOnce
    size: 2Gi

alertmanager:
  alertmanagerSpec:
    storage:
      volumeClaimTemplate:
        spec:
          storageClassName: "nfs-client"
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 2Gi

prometheus-node-exporter:
  enabled: true
  service:
    port: 9100

kube-state-metrics:
  enabled: true

prometheus-pushgateway:
  enabled: false

# -----------------------------
# Step 3: Deploy NGINX + Exporter
# -----------------------------
# kubectl apply -f nginx-pod.yaml
# kubectl apply -f nginx-service.yaml
# kubectl apply -f nginx-exporter-pod.yaml
# kubectl apply -f nginx-exporter-service.yaml

# -----------------------------
# Step 4: Access UIs
# -----------------------------
# Grafana:
#   kubectl get svc prometheus-grafana
#   http://<NodeIP>:<NodePort>
#   admin / admin
#
# Prometheus:
#   kubectl get svc prometheus-kube-prometheus-prometheus
#   http://<NodeIP>:<NodePort>

# -----------------------------
# Step 5: Verify Target
# -----------------------------
# In Prometheus UI → Status → Targets
# You should see a job named 'nginx-exporter' in UP state.
#
# In Grafana → Explore → Run a query such as:
#   nginx_connections_active
#
# Metrics should be visible if everything is working properly.

# -----------------------------
# Cleanup
# -----------------------------
# helm uninstall prometheus
# kubectl delete -f nginx-exporter-service.yaml
# kubectl delete -f nginx-exporter-pod.yaml
# kubectl delete -f nginx-service.yaml
# kubectl delete -f nginx-pod.yaml
# kubectl delete pvc --all
# =====================================================================

