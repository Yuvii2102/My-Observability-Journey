# Day 3 – Prometheus Architecture, PromQL and Grafana Practical

## Objective

In this practical, I learned how Prometheus collects Kubernetes metrics, how Kube State Metrics exposes Kubernetes resource information, and how PromQL helps us query and analyse those metrics.

I also explored Grafana's prebuilt Kubernetes dashboards and created my own dashboard to monitor container restarts.

### What I implemented

- Created an Amazon EKS cluster.
- Created a managed node group.
- Installed Prometheus and Grafana using Helm.
- Explored Kubernetes monitoring components.
- Queried Kubernetes metrics using PromQL.
- Created a crashing BusyBox Pod to observe container restarts.
- Practised PromQL filtering and aggregation.
- Explored prebuilt Grafana dashboards.
- Created a custom Grafana dashboard.
- Learned how ServiceMonitor helps Prometheus discover services.

---

# 1. Prerequisites

Before starting, I made sure the following tools were available on my Ubuntu EC2 instance:

- AWS CLI
- kubectl
- eksctl
- Helm
- curl

I checked the installed versions:

```bash
aws --version
kubectl version --client
eksctl version
helm version
```

I also made sure my AWS CLI was configured with credentials that had the required permissions to create EKS clusters and related AWS resources.

To check my AWS identity:

```bash
aws sts get-caller-identity
```

**Note:** These tools must be installed before starting. I used my Ubuntu EC2 instance as the terminal for this practical.

---

# 2. Create an EKS Cluster

Since I had deleted the resources after my previous practical, I created a new EKS cluster.

I used the following command:

```bash
eksctl create cluster \
  --name=observability \
  --region=us-east-1 \
  --zones=us-east-1a,us-east-1b \
  --without-nodegroup
```

### Explanation

- `eksctl create cluster` creates an EKS cluster.
- `--name` specifies the cluster name.
- `--region` specifies the AWS region.
- `--zones` specifies the availability zones.
- `--without-nodegroup` creates the control plane without worker nodes.

I created the control plane first because I wanted to configure the managed node group separately.

I waited until the cluster creation process completed successfully.

---

# 3. Associate IAM OIDC Provider

Next, I associated an IAM OIDC provider with my EKS cluster.

```bash
eksctl utils associate-iam-oidc-provider \
  --region us-east-1 \
  --cluster observability \
  --approve
```

### Explanation

OIDC stands for OpenID Connect.

Associating an OIDC provider allows Kubernetes service accounts to use IAM roles through mechanisms such as IAM Roles for Service Accounts (IRSA).

The `--approve` flag automatically approves the operation.

---

# 4. Create a Managed Node Group

Next, I created a managed node group with two worker nodes.

```bash
eksctl create nodegroup \
  --cluster=observability \
  --region=us-east-1 \
  --name=observability-ng-private \
  --node-type=t3.medium \
  --nodes-min=2 \
  --nodes-max=3 \
  --node-volume-size=20 \
  --managed \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --appmesh-access \
  --alb-ingress-access \
  --node-private-networking
```

### Explanation

| Option | Purpose |
|---|---|
| `--cluster` | Specifies the EKS cluster |
| `--region` | Specifies the AWS region |
| `--name` | Sets the node group name |
| `--node-type` | Selects the EC2 instance type |
| `--nodes-min` | Minimum number of worker nodes |
| `--nodes-max` | Maximum number of worker nodes |
| `--node-volume-size` | Sets the worker node volume size |
| `--managed` | Creates an EKS managed node group |
| `--asg-access` | Adds Auto Scaling group permissions |
| `--external-dns-access` | Adds ExternalDNS-related permissions |
| `--full-ecr-access` | Adds Amazon ECR access |
| `--appmesh-access` | Adds App Mesh-related permissions |
| `--alb-ingress-access` | Adds ALB Ingress-related permissions |
| `--node-private-networking` | Places worker nodes in private subnets |

I used `t3.medium` instances with 20 GB storage.

The minimum number of nodes was two and the maximum was three.

**Important:** Private worker nodes need suitable networking and outbound connectivity to download container images and communicate with AWS services. The VPC configuration must provide this connectivity.

---

# 5. Configure kubectl

After the node group was created, I configured kubectl to communicate with my EKS cluster.

```bash
aws eks update-kubeconfig \
  --name observability \
  --region us-east-1
```

### Verify the cluster

```bash
kubectl get nodes
```

Expected result:

I should see two worker nodes with the status `Ready`.

I also checked the Kubernetes namespaces:

```bash
kubectl get namespaces
```

This confirmed that my Kubernetes cluster was accessible.

---

# 6. Add the Prometheus Community Helm Repository

I used Helm to install the Prometheus monitoring stack.

First, I added the Prometheus Community repository:

```bash
helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts
```

Then I updated the Helm repositories:

```bash
helm repo update
```

### Explanation

Helm is a package manager for Kubernetes.

The Prometheus Community repository provides Helm charts for installing Prometheus-related components.

---

# 7. Create the Monitoring Namespace

I created a separate namespace to keep all monitoring resources organised.

```bash
kubectl create namespace monitoring
```

To verify:

```bash
kubectl get namespaces
```

I confirmed that the `monitoring` namespace was created successfully.

---

# 8. Download the Custom Prometheus Configuration

My teacher provided a custom configuration file for the kube-prometheus-stack installation.

I created a directory for the practical:

```bash
mkdir -p ~/day-2
cd ~/day-2
```

I downloaded the YAML file:

```bash
curl -L \
https://raw.githubusercontent.com/iam-veeramalla/observability-zero-to-hero/main/day-2/custom_kube_prometheus_stack.yml \
-o custom_kube_prometheus_stack.yml
```

To verify that the file was downloaded:

```bash
ls -lh custom_kube_prometheus_stack.yml
```

I could also inspect the configuration:

```bash
cat custom_kube_prometheus_stack.yml
```

### Why did I use this file?

The YAML file contains custom Helm values used to configure the monitoring stack.

Instead of installing the chart with its default configuration, I used my teacher's provided configuration.

**Configuration source:**

Teacher's GitHub repository:

`observability-zero-to-hero/day-2/custom_kube_prometheus_stack.yml`

---

# 9. Install Prometheus and Grafana Using Helm

I installed the kube-prometheus-stack chart in the monitoring namespace.

```bash
helm install monitoring \
  prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f ./custom_kube_prometheus_stack.yml
```

### Explanation

- `helm install` installs a Helm release.
- `monitoring` is the release name.
- `prometheus-community/kube-prometheus-stack` is the Helm chart.
- `-n monitoring` specifies the namespace.
- `-f` applies my custom YAML configuration.

The stack installs several monitoring components, including:

- Prometheus
- Grafana
- Alertmanager
- Prometheus Operator
- Node Exporter
- Kube State Metrics

---

# 10. Verify Monitoring Components

After the installation completed, I checked the Pods:

```bash
kubectl get pods -n monitoring
```

To check all the resources:

```bash
kubectl get all -n monitoring
```

I waited until the Pods showed the `Running` status and their containers were ready.

My monitoring namespace contained components such as:

| Component | Purpose |
|---|---|
| Prometheus | Collects and stores metrics |
| Grafana | Visualises monitoring data |
| Alertmanager | Handles alerts |
| Node Exporter | Collects node-level metrics |
| Kube State Metrics | Exposes Kubernetes object-state metrics |
| Prometheus Operator | Manages Prometheus resources |

To continuously watch Pod status:

```bash
kubectl get pods -n monitoring -w
```

I pressed `Ctrl + C` after verifying that the Pods were running.

---

# 11. Explore Kubernetes Monitoring Services

Next, I checked the Services created inside the monitoring namespace.

```bash
kubectl get svc -n monitoring
```

Some important Services from my installation were:

| Service | Port | Purpose |
|---|---:|---|
| `monitoring-grafana` | 80 | Grafana web interface |
| `monitoring-kube-state-metrics` | 8080 | Kube State Metrics endpoint |
| `monitoring-prometheus-node-exporter` | 9100 | Node Exporter endpoint |
| `prometheus-operated` | 9090 | Prometheus access |
| `monitoring-kube-prometheus-alertmanager` | 9093 | Alertmanager access |

### Understanding the two exporters

**Node Exporter**

Node Exporter collects infrastructure-level metrics from Kubernetes worker nodes.

Examples:

- CPU usage
- Memory usage
- Disk-related metrics
- System-level information

It runs on each worker node, normally as a DaemonSet.

Its metrics endpoint uses port `9100`.

**Kube State Metrics**

Kube State Metrics communicates with the Kubernetes API Server and exposes information about Kubernetes resources.

Examples:

- Pod status
- Deployment status
- ReplicaSet information
- Container restart counts
- ConfigMap information

Its metrics endpoint uses port `8080`.

**Important distinction:**

Node Exporter monitors the infrastructure.

Kube State Metrics monitors the state of Kubernetes objects.

---

# 12. Access the Prometheus Web Interface

I used port forwarding to access Prometheus from my browser.

```bash
kubectl port-forward \
  service/prometheus-operated \
  -n monitoring \
  9090:9090 \
  --address 0.0.0.0
```

I kept this terminal running.

### Configure the EC2 Security Group

I added an inbound rule:

| Setting | Value |
|---|---|
| Type | Custom TCP |
| Port | 9090 |
| Source | My IP |

I then opened the following address in my browser:

```text
http://<EC2-PUBLIC-IP>:9090
```

I replaced `<EC2-PUBLIC-IP>` with my actual EC2 public IPv4 address.

The Prometheus web interface opened successfully.

**Note:** Port forwarding must remain active while I access the interface.

---

# 13. Verify Prometheus Metrics

Inside the Prometheus interface, I used the Query page to explore metrics.

I first entered:

```promql
kube_pod_container_status_restarts_total
```

This metric displays the total number of container restarts reported by Kube State Metrics.

I clicked **Execute**.

Prometheus returned multiple result series containing labels such as:

- Namespace
- Pod
- Container
- Instance
- Job

### Filter Metrics Using Labels

Next, I filtered the metric by namespace:

```promql
kube_pod_container_status_restarts_total{namespace="default"}
```

This query returns restart metrics only for containers belonging to the `default` namespace.

I also learned that PromQL allows filtering using multiple labels.

For example:

```promql
kube_pod_container_status_restarts_total{
  namespace="default",
  pod="busybox-crash"
}
```

This query returns the restart count for the specific BusyBox Pod.

**Note:** If a namespace does not contain matching Pods or metrics, Prometheus may return an empty result.

---

# 14. Create a Crashing BusyBox Pod

To understand container restarts, I intentionally created a Pod whose container exits immediately.

I used:

```bash
kubectl run busybox-crash \
  --image=busybox \
  --restart=Always \
  -- /bin/sh -c "exit 1"
```

### Explanation

- `kubectl run` creates a Pod.
- `busybox-crash` is the Pod name.
- `--image=busybox` uses the BusyBox container image.
- `--restart=Always` allows Kubernetes to restart the container when it exits.
- `/bin/sh -c "exit 1"` runs a command that immediately exits with a failure status.

I checked the Pod:

```bash
kubectl get pods
```

The Pod entered `CrashLoopBackOff`.

This happened because the container repeatedly exited and Kubernetes attempted to restart it.

I checked the restart count using:

```bash
kubectl get pods
```

The `RESTARTS` column showed the number of times the container had restarted.

### Important command correction

I used `--restart=Always` intentionally.

Using `--restart=Never` would not demonstrate repeated container restarts because Kubernetes would not restart the failed container.

---

# 15. Monitor Container Restarts Using PromQL

I returned to the Prometheus browser interface.

I entered the following query:

```promql
kube_pod_container_status_restarts_total{
  namespace="default",
  pod="busybox-crash"
}
```

I clicked **Execute**.

Prometheus returned the restart count for my BusyBox container.

I also clicked the **Graph** tab to visualise the restart count over time.

As the container continued crashing, the restart count increased.

This helped me understand how Prometheus collects Kubernetes restart metrics through Kube State Metrics.

---

# 16. Explore Additional Kubernetes Metrics

Next, I explored a few more metrics from Kube State Metrics.

### Init Container Restart Count

```promql
kube_pod_init_container_status_restarts_total
```

This metric displays restart counts for init containers.

### ConfigMap Creation Timestamp

```promql
kube_configmap_created
```

This metric provides the creation timestamp of Kubernetes ConfigMaps.

I also explored the Prometheus metric autocomplete feature to discover available metrics.

This helped me understand how to find and query Kubernetes metrics without memorising every metric name.

---

# 17. PromQL Aggregation Functions

Next, I practised aggregation functions.

Aggregation helps combine multiple metric values into a single result or group results based on labels.

### A. Sum Function

```promql
sum(kube_pod_container_status_restarts_total)
```

This query calculates the total restart count across all matching time series.

In my practical, the result was `5` at the time I executed the query. The value can change because the BusyBox container continues restarting.

### B. Average Function

```promql
avg(kube_pod_container_status_restarts_total)
```

This calculates the average value across all matching time series.

My observed result was approximately `0.2083`.

### C. Grouping by Namespace

```promql
sum by (namespace) (
  kube_pod_container_status_restarts_total
)
```

This query groups the restart counts by Kubernetes namespace.

My observed results included:

| Namespace | Restart count |
|---|---:|
| kube-system | 0 |
| monitoring | 0 |
| default | 6 |

The values can change as the containers restart.

### What I learned

| Function | Purpose |
|---|---|
| `sum()` | Calculates the total |
| `avg()` | Calculates the average |
| `sum by (namespace)` | Calculates totals separately for each namespace |

---

# 18. Access Grafana

After exploring Prometheus, I accessed Grafana.

I opened a new Ubuntu terminal and ran:

```bash
kubectl port-forward \
  service/monitoring-grafana \
  -n monitoring \
  8081:80 \
  --address 0.0.0.0
```

I used port `8081` because port `8080` was already being used on my EC2 instance.

### Configure the EC2 Security Group

I added an inbound rule:

| Setting | Value |
|---|---|
| Type | Custom TCP |
| Port | 8081 |
| Source | My IP |

I opened Grafana in my browser:

```text
http://<EC2-PUBLIC-IP>:8081
```

Grafana opened successfully.

### Grafana Login Credentials

Username:

```text
admin
```

To retrieve the actual password from Kubernetes, I used:

```bash
kubectl get secret monitoring-grafana \
  -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 -d
echo
```

I used the password displayed by this command to log in.

**Note:** I retrieved the password from the Kubernetes Secret instead of assuming a default password.

---

# 19. Explore Prebuilt Grafana Dashboards

After logging in, I opened:

**Dashboards → Dashboards**

Grafana already provided several prebuilt Kubernetes dashboards.

Some dashboards I explored included:

- Alertmanager / Overview
- CoreDNS
- etcd
- Grafana Overview
- Kubernetes / API Server
- Kubernetes / Compute Resources / Cluster
- Kubernetes / Compute Resources / Pod

### A. Kubernetes Compute Resources / Cluster

I opened:

**Kubernetes / Compute Resources / Cluster**

This dashboard displayed information such as:

- CPU utilisation
- CPU requests
- CPU limits
- Memory utilisation
- Memory requests
- Memory limits
- CPU usage over time
- Namespace-level resource information

I observed CPU and memory usage for namespaces such as:

- default
- kube-system
- monitoring

This helped me understand how Grafana visualises Kubernetes resource metrics.

### B. Kubernetes Compute Resources / Pod

Next, I opened:

**Kubernetes / Compute Resources / Pod**

I first selected the `busybox-crash` Pod.

The dashboard showed `No data` for CPU Usage and CPU Throttling.

This was expected because the BusyBox container exits immediately and does not remain active long enough to produce meaningful CPU usage data.

I then changed the namespace to `monitoring` and selected my Grafana Pod.

The dashboard displayed CPU usage for the Grafana containers.

I could observe the CPU usage changing over time.

This helped me understand how Grafana provides Pod-level resource visualisation.

---

# 20. Create My Own Grafana Dashboard

After exploring the prebuilt dashboards, I created a custom dashboard to monitor Kubernetes container restarts.

### Step 1: Create a New Dashboard

I opened:

**Dashboards → New → New dashboard**

Then I clicked the option to add a panel.

### Step 2: Configure the Panel

I selected **Configure visualization**.

I changed the panel title to:

```text
Kubernetes Container Restart Count
```

I also selected Prometheus as the data source.

### Step 3: Enter the PromQL Query

I switched from Builder mode to Code mode.

I entered:

```promql
kube_pod_container_status_restarts_total{
  namespace="default",
  pod="busybox-crash"
}
```

Then I clicked **Run queries**.

### Step 4: Select Visualization

I selected:

**Time series**

This displayed the restart count as a graph over time.

The graph showed the number of times my BusyBox container had restarted.

Grafana also displayed a warning that the selected metric was a counter.

This was expected because the restart count is a cumulative counter.

### Step 5: Add Panel Description

I used the following description:

> Monitors Kubernetes container restarts using Prometheus and Grafana. Displays restart counts over time using PromQL queries and metrics collected by Kube State Metrics. Helps identify repeatedly crashing containers and understand their restart behaviour.

### Step 6: Save the Dashboard

I saved the panel and then saved the dashboard.

Dashboard name:

```text
Kubernetes Container Restart Monitoring
```

I verified that the dashboard appeared in my Grafana dashboard list.

My first custom Grafana dashboard was successfully created!

---

# 21. Understand ServiceMonitor

The final concept from my teacher's Day 3 lesson was ServiceMonitor.

### What is ServiceMonitor?

ServiceMonitor is a Kubernetes custom resource provided by the Prometheus Operator.

It tells Prometheus which Kubernetes Services it should monitor and scrape for metrics.

### Example

Imagine I have an e-commerce application with these services:

- Login Service
- Payment Service
- Logout Service
- Product Service
- Cart Service

Suppose only the Login Service and Payment Service expose application metrics through a `/metrics` endpoint.

I don't necessarily want Prometheus to scrape every service.

I can use ServiceMonitor to select the services that should be monitored.

### How ServiceMonitor Works

1. Developers instrument their applications with metrics.
2. Applications expose metrics through a `/metrics` endpoint.
3. Kubernetes Services provide access to those applications.
4. ServiceMonitor selects the required Services.
5. Prometheus discovers the selected Services and scrapes their metrics.
6. Prometheus stores the collected metrics in its time-series database.

### Important Components

| Component | Responsibility |
|---|---|
| Node Exporter | Collects node-level metrics |
| Kube State Metrics | Exposes Kubernetes object-state metrics |
| ServiceMonitor | Selects Services for Prometheus scraping |
| Prometheus | Scrapes and stores metrics |
| Grafana | Visualises metrics |

**Important:** ServiceMonitor does not generate application metrics. Developers must instrument their applications and expose a metrics endpoint.

My teacher introduced this concept in Day 3. The practical implementation of custom application metrics and ServiceMonitor was part of the upcoming lesson, so I kept this section theoretical.

---

# 22. Cleanup – Delete AWS Resources

After completing the practical, I cleaned up the EKS environment to avoid unnecessary AWS charges.

Since I had created the EKS cluster specifically for this practical, I deleted it.

**Important:** I ran the following command from my Ubuntu EC2 instance, not from the Prometheus or Grafana browser.

```bash
eksctl delete cluster \
  --name observability \
  --region us-east-1
```

I waited until the deletion process completed.

### Verify the Cluster Deletion

```bash
eksctl get cluster --region us-east-1
```

I checked the AWS EKS console to confirm that the cluster was deleted.

I also checked the EC2 console for any remaining worker instances.

### Additional Cleanup

I reviewed the following resources in AWS:

- EKS clusters
- EC2 instances
- EBS volumes
- NAT Gateways
- Elastic IP addresses
- Load Balancers
- Security Group inbound rules

I removed temporary inbound rules used for this practical, including ports `9090`, `8081` and any other temporary monitoring access rules, where no longer needed.

**Important AWS cost note:**

Deleting the EKS cluster does not automatically guarantee that every separately created AWS resource has been removed. I checked for remaining resources to avoid unexpected charges.

I kept my separate Ubuntu EC2 instance only if I needed it for other projects. Otherwise, I stopped or terminated it as appropriate.

---

# 23. Important Commands – Quick Revision

These are the main commands I used during the practical.

### EKS Cluster Setup

```bash
eksctl create cluster \
  --name=observability \
  --region=us-east-1 \
  --zones=us-east-1a,us-east-1b \
  --without-nodegroup
```

```bash
eksctl utils associate-iam-oidc-provider \
  --region us-east-1 \
  --cluster observability \
  --approve
```

```bash
eksctl create nodegroup \
  --cluster=observability \
  --region=us-east-1 \
  --name=observability-ng-private \
  --node-type=t3.medium \
  --nodes-min=2 \
  --nodes-max=3 \
  --node-volume-size=20 \
  --managed \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --appmesh-access \
  --alb-ingress-access \
  --node-private-networking
```

```bash
aws eks update-kubeconfig \
  --name observability \
  --region us-east-1
```

### Install Monitoring Stack

```bash
helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts
```

```bash
helm repo update
```

```bash
kubectl create namespace monitoring
```

```bash
mkdir -p ~/day-2
cd ~/day-2
```

```bash
curl -L \
https://raw.githubusercontent.com/iam-veeramalla/observability-zero-to-hero/main/day-2/custom_kube_prometheus_stack.yml \
-o custom_kube_prometheus_stack.yml
```

```bash
helm install monitoring \
  prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f ./custom_kube_prometheus_stack.yml
```

### Verify Kubernetes Resources

```bash
kubectl get nodes
```

```bash
kubectl get namespaces
```

```bash
kubectl get pods -n monitoring
```

```bash
kubectl get svc -n monitoring
```

```bash
kubectl get all -n monitoring
```

### Port Forwarding

Prometheus:

```bash
kubectl port-forward \
  service/prometheus-operated \
  -n monitoring \
  9090:9090 \
  --address 0.0.0.0
```

Grafana:

```bash
kubectl port-forward \
  service/monitoring-grafana \
  -n monitoring \
  8081:80 \
  --address 0.0.0.0
```

### Create a Crashing Pod

```bash
kubectl run busybox-crash \
  --image=busybox \
  --restart=Always \
  -- /bin/sh -c "exit 1"
```

```bash
kubectl get pods
```

### Retrieve Grafana Password

```bash
kubectl get secret monitoring-grafana \
  -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 -d
echo
```

### Delete the EKS Cluster

```bash
eksctl delete cluster \
  --name observability \
  --region us-east-1
```

---

# 24. PromQL Queries – Quick Revision

| Purpose | PromQL Query |
|---|---|
| All container restart counts | `kube_pod_container_status_restarts_total` |
| Restart counts in default namespace | `kube_pod_container_status_restarts_total{namespace="default"}` |
| Restart count of BusyBox Pod | `kube_pod_container_status_restarts_total{namespace="default",pod="busybox-crash"}` |
| Init container restart counts | `kube_pod_init_container_status_restarts_total` |
| ConfigMap creation timestamp | `kube_configmap_created` |
| Total restart count | `sum(kube_pod_container_status_restarts_total)` |
| Average restart count | `avg(kube_pod_container_status_restarts_total)` |
| Restart count grouped by namespace | `sum by (namespace) (kube_pod_container_status_restarts_total)` |

---

# 25. What I Learned

In this practical, I learned how Prometheus collects metrics from Kubernetes monitoring components and how Kube State Metrics exposes Kubernetes resource information.

I practised PromQL queries to filter, aggregate and analyse metrics.

I also created a crashing BusyBox Pod and monitored its restart count using Prometheus.

After that, I explored Grafana's prebuilt Kubernetes dashboards and created my own custom dashboard to visualise container restarts.

Finally, I learned how ServiceMonitor helps Prometheus discover and scrape selected Kubernetes Services.

This practical helped me understand how Prometheus and Grafana work together to monitor Kubernetes environments.

---

# 26. References

I followed my teacher Abhishek Veeramalla's Observability Zero to Hero series.

**Teacher's GitHub Repository:**

https://github.com/iam-veeramalla/observability-zero-to-hero

**Custom Prometheus Configuration:**

https://raw.githubusercontent.com/iam-veeramalla/observability-zero-to-hero/main/day-2/custom_kube_prometheus_stack.yml

**Prometheus Community Helm Charts:**

https://github.com/prometheus-community/helm-charts

**Kube Prometheus:**

https://github.com/prometheus-operator/kube-prometheus

**Official Documentation:**

- Amazon EKS: https://docs.aws.amazon.com/eks/
- Kubernetes: https://kubernetes.io/docs/
- Prometheus: https://prometheus.io/docs/
- PromQL: https://prometheus.io/docs/prometheus/latest/querying/basics/
- Grafana: https://grafana.com/docs/
- Helm: https://helm.sh/docs/
