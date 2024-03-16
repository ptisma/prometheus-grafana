Steps to deploy Prometheus:
Create a Kubernetes namespace for all our monitoring components:
kubectl create namespace monitoring

Create an RBAC policy with read access to required API groups and bind the policy to the monitoring namespace:
kubectl apply -f clusterRole.yaml

Create a deployment on monitoring namespace:
kubectl create  -f deployment.yaml 

To access the Prometheus dashboard over a IP or a DNS name, you need to expose it as a Kubernetes service:
kubectl create  -f service.yaml 

Expose with minikube:
minikube service prometheus-service -n monitoring --url

Create Deployments for a test to see the pod metrics:
kubectl apply -f target-pod.yaml



Steps to deploy deploy kube-state-metrics (they follow the pretty much similiar pattern to Prometheus):
kubectl create namespace monitoring

YAML manifest sets up RBAC (Role-Based Access Control) for the kube-state-metrics exporter in a Kubernetes cluster. 
Permissions include listing and watching resources such as configmaps, secrets, nodes, pods, services, etc.:
kubectl apply -f roles.yaml

YAML manifest sets up the kube-state-metrics deployment.It's a service that talks to the Kubernetes API server to get all the details about all the API objects like deployments, pods, daemonsets, Statefulsets, etc. and then produces the metrics in Prometheus format on /metrics endpoint (port 8080), it's own metrics are on port 8081.
kubectl apply -f deployment.yaml

To finally expose the deployment through service run:
kubectl apply -f service.yaml


Steps to deploy grafana:
Create the configuration for grafana server(add Prometheus server as data source):
kubectl apply -f config-map.yaml

Create the deployment:
kubectl apply -f config-map.yaml

Expose with minikube and login to Grafana:
minikube service grafana -n monitoring --url

Download the Grafana templates to monitor the deployments:
https://grafana.com/grafana/dashboards/8588-1-kubernetes-deployment-statefulset-daemonset-metrics/

https://grafana.com/grafana/dashboards/741-deployment-metrics/

https://grafana.com/grafana/dashboards/20372-kubernetes-deployment-cpu-and-memory-metrics/



Steps to deploy node-exporter:
Create the daemon-set, one pod per node which captures the metrics exposed by kernel.
kubectl apply -f daemon-set.yaml

Create the service which exposes the daemon set through service to prometheus:
kubectl apply -f service.yaml

Add the scrape config to prometheus to enable node metrics:
```
- job_name: 'node-exporter'
        kubernetes_sd_configs:
          - role: endpoints
        relabel_configs:
        - source_labels: [__meta_kubernetes_endpoints_name]
          regex: 'node-exporter'
          action: keep
```

Install a grafana dashboard for node metrics:
https://grafana.com/grafana/dashboards/1860-node-exporter-full/





Steps to deploy alert-manager:

Create the config for alert-manager (here we are going to define our receivers, routes etc.):
kubectl apply -f config-map.yaml

Since we are using the email, gmail in this case, as a receiver we have to setup this using Gmail -> Settings -> App password to give the alert-manager app credentials to use the email on our behalf.

Create deployment and service:
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

Expose with minikube:
minikube service alertmanager -n monitoring --url

Edit the prometheus config to setup some rules on prometheus:
More here https://samber.github.io/awesome-prometheus-alerts/rules.html

Edit the prometheus config to connect to alert-manager service:
```
alertmanagers:
      - scheme: http
        static_configs:
        - targets:
          - "alertmanager.monitoring.svc:9093"
```
Keep in mind since we are not using the operator and its auto-watch mechanism, once we change the Prometheus config map the pod restart is required.





