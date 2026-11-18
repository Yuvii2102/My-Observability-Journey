# Day 2 – Installing Prometheus, Grafana and Alertmanager on AWS EKS

## Introduction

In this practical, I installed Prometheus, Grafana and Alertmanager on an AWS EKS cluster using Helm.

I used the `kube-prometheus-stack` Helm chart provided by the Prometheus Community.

The main objective of this practical was to set up a monitoring system for Kubernetes and access the monitoring dashboards.

### Tools and Technologies Used

- AWS EKS
- AWS EC2
- AWS CLI
- eksctl
- kubectl
- Helm
- Prometheus
- Grafana
- Alertmanager
- Node Exporter
- Kube State Metrics

---

# Step 1: Create an EKS Cluster

First, I created an EKS cluster named `observability` in the AWS US East (N. Virginia) region.

I used eksctl to create the cluster without a node group.

### Command

```bash
eksctl create cluster \
  --name=observability \
  --region=us-east-1 \
  --zones=us-east-1a,us-east-1b \
  --without-nodegroup
```

### Explanation

- `eksctl create cluster` – Creates an EKS cluster.
- `--name` – Specifies the cluster name.
- `--region` – Specifies the AWS region.
- `--zones` – Specifies the availability zones.
- `--without-nodegroup` – Creates the cluster without worker nodes.

### What happened?

AWS started creating the EKS control plane and the required infrastructure using CloudFormation.

After the creation process was completed, I received:

```text
all EKS cluster resources for "observability" have been created

EKS cluster "observability" in "us-east-1" region is ready
```

My EKS cluster was successfully created.

---

# Step 2: Associate IAM OIDC Provider

Next, I associated an IAM OIDC provider with my EKS cluster.

### What is OIDC?

OIDC stands for OpenID Connect.

It allows Kubernetes Service Accounts to obtain AWS permissions through IAM roles using identity federation.

This is useful when Kubernetes workloads need to access AWS services securely.

### Command

```bash
eksctl utils associate-iam-oidc-provider \
  --region us-east-1 \
  --cluster observability \
  --approve
```

### Explanation

- `associate-iam-oidc-provider` – Associates an OIDC provider with the cluster.
- `--region` – Specifies the AWS region.
- `--cluster` – Specifies the EKS cluster name.
- `--approve` – Automatically approves the creation.

### Expected output

```text
will create IAM Open ID Connect provider for cluster "observability"

created IAM Open ID Connect provider for cluster "observability"
```

My OIDC provider was successfully created.

---

# Step 3: Create a Managed Node Group

An EKS control plane manages the Kubernetes cluster, but we also need worker nodes to run our applications and Pods.

So, I created a managed node group with two EC2 worker nodes.

### Command

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

- `--cluster` – Specifies the EKS cluster.
- `--region` – Specifies the AWS region.
- `--name` – Specifies the node group name.
- `--node-type` – Specifies the EC2 instance type.
- `--nodes-min` – Sets the minimum number of nodes.
- `--nodes-max` – Sets the maximum number of nodes.
- `--node-volume-size` – Sets the root volume size.
- `--managed` – Creates a managed node group.
- `--asg-access` – Configures Auto Scaling Group access.
- `--external-dns-access` – Adds External DNS-related IAM permissions.
- `--full-ecr-access` – Adds Amazon ECR access permissions.
- `--appmesh-access` – Adds App Mesh-related permissions.
- `--alb-ingress-access` – Adds ALB Ingress-related permissions.
- `--node-private-networking` – Places worker nodes in private subnets.

### What happened?

eksctl created the managed node group and launched two EC2 worker nodes.

The output confirmed:

```text
nodegroup "observability-ng-private" has 2 node(s)

node "ip-192-168-121-162.ec2.internal" is ready

node "ip-192-168-76-252.ec2.internal" is ready
```

My managed node group was successfully created.

---

# Step 4: Update kubeconfig

Next, I updated the kubeconfig file so that kubectl could communicate with my EKS cluster.

### Command

```bash
aws eks update-kubeconfig \
  --name observability \
  --region us-east-1
```

### Explanation

- `aws eks update-kubeconfig` – Updates the Kubernetes configuration file.
- `--name` – Specifies the EKS cluster.
- `--region` – Specifies the AWS region.

The kubeconfig file stores the connection information required by kubectl to communicate with the cluster.

### Verify the worker nodes

```bash
kubectl get nodes
```

### Output

```text
NAME                            STATUS   ROLES    AGE   VERSION
ip-192-168-121-162.ec2.internal Ready    <none>   ...   v1.34.12
ip-192-168-76-252.ec2.internal  Ready    <none>   ...   v1.34.12
```

Both worker nodes were showing `Ready`.

My EKS cluster was ready for the monitoring installation.

---

# Step 5: Add the Prometheus Helm Repository

Before installing Prometheus, I added the Prometheus Community Helm repository.

### What is Helm?

Helm is a package manager for Kubernetes.

It helps us install, configure and manage applications in a Kubernetes cluster using Helm charts.

### Command 1: Add the repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

### Command 2: Update the repository

```bash
helm repo update
```

### Explanation

- `helm repo add` – Adds a Helm chart repository.
- `prometheus-community` – Repository name.
- The URL points to the Prometheus Community Helm charts.
- `helm repo update` – Updates the chart information from the configured repositories.

### Expected output

```text
"prometheus-community" has been added to your repositories

Successfully got an update from the "prometheus-community" chart repository

Update Complete. Happy Helming!
```

The Prometheus Community repository was successfully added.

---

# Step 6: Create a Monitoring Namespace

Next, I created a separate Kubernetes namespace called `monitoring`.

### What is a Namespace?

A namespace helps organize and separate Kubernetes resources inside a cluster.

I used a separate namespace to keep all my monitoring components organized.

### Command

```bash
kubectl create namespace monitoring
```

### Verify the namespace

```bash
kubectl get namespaces
```

### Expected output

```text
NAME              STATUS
default           Active
kube-node-lease   Active
kube-public       Active
kube-system       Active
monitoring        Active
```

My monitoring namespace was successfully created.

---

# Step 7: Download the Custom Prometheus Configuration File

Before installing the monitoring stack, I downloaded the custom configuration file provided by my teacher.

File name:

`custom_kube_prometheus_stack.yml`

### Create the Day 2 directory

```bash
mkdir -p day-2
```

### Navigate to the directory

```bash
cd ~/day-2
```

### Download the configuration file

```bash
curl -L https://raw.githubusercontent.com/iam-veeramalla/observability-zero-to-hero/main/day-2/custom_kube_prometheus_stack.yml -o custom_kube_prometheus_stack.yml
```

### Verify the file

```bash
cat custom_kube_prometheus_stack.yml
```

### What does this file contain?

The custom YAML file contains configuration values for Grafana and Alertmanager.

Some important configurations include:

**Grafana password**

```yaml
grafana:
  adminPassword: prom-operator
```

**Alertmanager configuration**

```yaml
alertmanager:
  alertmanagerSpec:
```

**Alertmanager configuration selector**

```yaml
alertmanagerConfigSelector:
  matchLabels:
    release: monitoring
```

**Alertmanager replicas**

```yaml
replicas: 2
```

**Alertmanager matcher strategy**

```yaml
alertmanagerConfigMatcherStrategy:
  type: None
```

I used the same configuration file provided in my teacher's GitHub repository instead of creating a new one.

---

# Step 8: Install kube-prometheus-stack Using Helm

Now I installed the complete monitoring stack using the Prometheus Community Helm chart.

### Command

```bash
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring -f ./custom_kube_prometheus_stack.yml
```

### Explanation

- `helm install` – Installs a Helm chart.
- `monitoring` – Name of the Helm release.
- `prometheus-community/kube-prometheus-stack` – Helm chart used for installation.
- `-n monitoring` – Installs the resources in the monitoring namespace.
- `-f` – Uses the custom YAML configuration file.

### What does kube-prometheus-stack install?

The chart installs the following monitoring components:

| Component | Purpose |
|---|---|
| Prometheus | Collects and stores metrics |
| Grafana | Visualizes metrics using dashboards |
| Alertmanager | Handles alerts |
| Node Exporter | Collects node-level metrics |
| Kube State Metrics | Exposes Kubernetes resource information |
| Prometheus Operator | Manages Prometheus-related Kubernetes resources |

### Expected output

```text
NAME: monitoring
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
```

My monitoring stack was successfully installed.

---

# Step 9: Verify the Monitoring Components

After installing the monitoring stack, I checked the resources inside the monitoring namespace.

### Command

```bash
kubectl get all -n monitoring
```

### Explanation

- `kubectl get all` – Displays the main Kubernetes resources.
- `-n monitoring` – Displays resources from the monitoring namespace.

I checked the Pods, Services, Deployments, ReplicaSets, DaemonSets and StatefulSets.

### Check Pod status

```bash
kubectl get pods -n monitoring
```

Initially, Grafana was showing `2/3 Running` because one of its containers was not ready yet.

I waited and used the following command to watch the Pod status.

```bash
kubectl get pods -n monitoring -w
```

The `-w` option continuously watches the resource status.

After a few minutes, all the monitoring Pods were running.

### Final Pod status

| Component | Ready status |
|---|---|
| Prometheus | 2/2 |
| Grafana | 3/3 |
| Alertmanager | 2/2 each |
| Node Exporter | 1/1 each |
| Kube State Metrics | 1/1 |
| Prometheus Operator | 1/1 |

I pressed `Ctrl + C` to stop watching the Pod status.

All the monitoring components were running successfully.

---

# Step 10: Access the Prometheus Dashboard

After verifying the monitoring components, I accessed the Prometheus dashboard using port forwarding.

### What is Port Forwarding?

Port forwarding allows me to access a Kubernetes Service through a port on my EC2 instance.

### Command

```bash
kubectl port-forward service/prometheus-operated -n monitoring 9090:9090 --address 0.0.0.0
```

### Explanation

- `kubectl port-forward` – Forwards a local port to a Kubernetes Service.
- `service/prometheus-operated` – Prometheus Service.
- `-n monitoring` – Specifies the monitoring namespace.
- `9090:9090` – Maps port 9090 on the EC2 instance to port 9090 of the Service.
- `--address 0.0.0.0` – Makes the port-forward listener available on all network interfaces of the EC2 instance.

### Expected output

```text
Forwarding from 0.0.0.0:9090 -> 9090
```

### EC2 Security Group Configuration

Since I was using an EC2 instance, I added an inbound rule to its Security Group.

| Field | Value |
|---|---|
| Type | Custom TCP |
| Protocol | TCP |
| Port | 9090 |
| Source | My IP |

I opened the following URL in my browser:

```text
http://<EC2-PUBLIC-IP>:9090
```

The Prometheus dashboard opened successfully.

I could see the Query, Alerts and Status sections.

---

# Step 11: Access the Grafana Dashboard

Next, I accessed Grafana using port forwarding.

### Command

```bash
kubectl port-forward service/monitoring-grafana -n monitoring 8080:80 --address 0.0.0.0
```

### Explanation

- `service/monitoring-grafana` – Grafana Kubernetes Service.
- `-n monitoring` – Specifies the monitoring namespace.
- `8080:80` – Maps EC2 port 8080 to Grafana Service port 80.
- `--address 0.0.0.0` – Allows access through the EC2 network interfaces.

### Expected output

```text
Forwarding from 0.0.0.0:8080 -> 80
```

### EC2 Security Group Configuration

I added another inbound rule.

| Field | Value |
|---|---|
| Type | Custom TCP |
| Protocol | TCP |
| Port | 8080 |
| Source | My IP |

### Access Grafana

I opened:

```text
http://<EC2-PUBLIC-IP>:8080
```

### Grafana Login Credentials

| Field | Value |
|---|---|
| Username | admin |
| Password | prom-operator |

These credentials came from my teacher's custom YAML file.

I logged in successfully and the Grafana home dashboard opened.

---

# Step 12: Access the Alertmanager Dashboard

Finally, I accessed Alertmanager using port forwarding.

### Command

```bash
kubectl port-forward service/alertmanager-operated -n monitoring 9093:9093 --address 0.0.0.0
```

### Explanation

- `service/alertmanager-operated` – Alertmanager Kubernetes Service.
- `-n monitoring` – Specifies the monitoring namespace.
- `9093:9093` – Maps EC2 port 9093 to Alertmanager port 9093.
- `--address 0.0.0.0` – Allows access through the EC2 network interfaces.

### Expected output

```text
Forwarding from 0.0.0.0:9093 -> 9093
```

### EC2 Security Group Configuration

I added an inbound rule for Alertmanager.

| Field | Value |
|---|---|
| Type | Custom TCP |
| Protocol | TCP |
| Port | 9093 |
| Source | My IP |

### Access Alertmanager

I opened:

```text
http://<EC2-PUBLIC-IP>:9093
```

The Alertmanager dashboard opened successfully.

I could see the Alerts, Silences, Status and Settings sections.

---

# Step 13: Cleanup

After completing the practical, I used the cleanup commands provided by my teacher to remove the monitoring resources and EKS cluster.

**Important:** These commands delete the monitoring installation and AWS infrastructure. I should only execute them when I no longer need this environment.

### Step 1: Uninstall the Helm release

```bash
helm uninstall monitoring --namespace monitoring
```

This removes the `monitoring` Helm release and its chart-managed resources.

### Step 2: Delete the monitoring namespace

```bash
kubectl delete ns monitoring
```

This deletes the monitoring namespace and its remaining resources.

### Step 3: Delete the EKS cluster

```bash
eksctl delete cluster --name observability
```

This deletes the EKS cluster and its managed node group.

I also need to remove the temporary inbound Security Group rules for ports 9090, 8080 and 9093 if they are no longer required.

---

# Final Result

I successfully installed and accessed the complete monitoring stack on AWS EKS using Helm.

The three dashboards were accessible through my EC2 instance.

| Tool | Purpose | Port |
|---|---|---|
| Prometheus | Metrics collection and querying | 9090 |
| Grafana | Metrics visualization | 8080 |
| Alertmanager | Alert management | 9093 |

## What I Learned

1. How to create an EKS cluster using eksctl.
2. How to associate an IAM OIDC provider.
3. How to create a managed node group.
4. How to configure kubectl to communicate with EKS.
5. How to add and update Helm repositories.
6. How to create Kubernetes namespaces.
7. How to download and use a custom Helm values file.
8. How to install kube-prometheus-stack using Helm.
9. How to verify monitoring Pods and Services.
10. How to access Prometheus, Grafana and Alertmanager using port forwarding.
11. How to clean up the monitoring resources and EKS cluster.

## My Understanding

In this practical, I learned how to set up monitoring for a Kubernetes cluster using Prometheus, Grafana and Alertmanager.

Prometheus collects and stores metrics. Grafana helps us visualize those metrics using dashboards. Alertmanager handles alerts generated from Prometheus.

Using Helm made it easier to install all these monitoring components in my EKS cluster.

This practical helped me understand how monitoring tools are installed and accessed in a real Kubernetes environment.

---

## References and Sources

I followed my teacher Abhishek Veeramalla's Observability Zero to Hero series and used the commands and configuration from the Day 2 practical.

**GitHub Repository:**

https://github.com/iam-veeramalla/observability-zero-to-hero

**Custom Configuration File:**

https://github.com/iam-veeramalla/observability-zero-to-hero/blob/main/day-2/custom_kube_prometheus_stack.yml

**Prometheus Community Helm Charts:**

https://github.com/prometheus-community/helm-charts

**Note:** The `--address 0.0.0.0` option was added to the three port-forward commands because I was accessing the dashboards through an EC2 instance. I also configured the corresponding EC2 Security Group inbound rules with My IP as the source.
