# Day 5 – Logging with EFK Stack

## 1. Connect to the Kubernetes Cluster

First, I will connect to my EKS cluster.

```bash
aws eks update-kubeconfig --region ap-south-1 --name observability-one
```

Verify the connection:

```bash
kubectl get nodes
```

Check the existing pods:

```bash
kubectl get pods -A
```

## 2. Create IAM Role for EBS CSI Driver

**Why?** Elasticsearch needs persistent storage. Since I am using AWS EBS volumes, the EKS cluster needs permission to create and manage those volumes.

Create the IAM OIDC provider:

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster observability-one \
  --region ap-south-1 \
  --approve
```

Create the IAM service account:

```bash
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster observability-one \
  --region ap-south-1 \
  --role-name AmazonEKS_EBS_CSI_DriverRole \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve
```

Retrieve the IAM role ARN:

```bash
aws iam get-role \
  --role-name AmazonEKS_EBS_CSI_DriverRole \
  --query 'Role.Arn' \
  --output text
```

## 3. Install AWS EBS CSI Driver

The EBS CSI Driver allows Kubernetes to create and attach AWS EBS volumes.

Install the EKS add-on:

```bash
aws eks create-addon \
  --cluster-name observability-one \
  --addon-name aws-ebs-csi-driver \
  --region ap-south-1
```

Verify the installation:

```bash
kubectl get pods -n kube-system
```

Check the CSI driver:

```bash
kubectl get csidrivers
```

## 4. Create Logging Namespace

I will create a separate namespace to deploy all EFK components.

```bash
kubectl create namespace logging
```

Verify:

```bash
kubectl get namespaces
```

## 5. Install Elasticsearch Using Helm

First, I will add the Elasticsearch Helm repository.

```bash
helm repo add elastic https://helm.elastic.co
```

Update the repository:

```bash
helm repo update
```

Install Elasticsearch with two replicas and AWS EBS persistent storage using the gp2 StorageClass:

```bash
helm install elasticsearch elastic/elasticsearch \
  --namespace logging \
  --set replicas=2 \
  --set volumeClaimTemplate.storageClassName=gp2
```

Check Elasticsearch pods:

```bash
kubectl get pods -n logging
```

Check the services:

```bash
kubectl get svc -n logging
```

Check persistent volume claims:

```bash
kubectl get pvc -n logging
```

The PVC should eventually become Bound.

## 6. Retrieve Elasticsearch Credentials

Fluent Bit needs Elasticsearch authentication credentials to forward logs.

Retrieve the username:

```bash
echo elastic
```

Retrieve the password:

```bash
kubectl get secret elasticsearch-master-credentials \
  -n logging \
  -o jsonpath='{.data.password}' | base64 --decode
```

Copy and save the password. I will use it while configuring Fluent Bit.

## 7. Install Kibana Using Helm

Add the Elasticsearch Helm repository if it has not already been added:

```bash
helm repo add elastic https://helm.elastic.co
helm repo update
```

Install Kibana:

```bash
helm install kibana elastic/kibana \
  --namespace logging \
  --set service.type=LoadBalancer
```

Verify:

```bash
kubectl get pods -n logging
```

Check the service:

```bash
kubectl get svc -n logging
```

Copy the external LoadBalancer address of Kibana.

Open the following in my browser:

```text
http://<KIBANA-LOADBALANCER-ADDRESS>:5601
```

Login credentials:

```text
Username: elastic
Password: <ELASTICSEARCH-PASSWORD>
```

## 8. Install Fluent Bit Using Helm

Fluent Bit collects container logs from every Kubernetes node and forwards them to Elasticsearch.

Add the Fluent Bit repository:

```bash
helm repo add fluent https://fluent.github.io/helm-charts
```

Update the repository:

```bash
helm repo update
```

Before installing Fluent Bit, I need to download the instructor's Day 5 configuration files.

GitHub repository:

https://github.com/iam-veeramalla/observability-zero-to-hero

Clone the repository:

```bash
git clone https://github.com/iam-veeramalla/observability-zero-to-hero.git
```

Navigate to the Day 5 folder:

```bash
cd observability-zero-to-hero/day-5
```

Open the Fluent Bit configuration file:

```bash
vim fluent-bit.yaml
```

Find the Elasticsearch output configuration.

Update the following credentials in BOTH Elasticsearch output sections:

```yaml
HTTP_User: elastic
HTTP_Passwd: "<ELASTICSEARCH-PASSWORD>"
```

Make sure the password is correct in both locations.

Save the file:

```text
Esc
:wq
Enter
```

Install Fluent Bit:

```bash
helm install fluent-bit fluent/fluent-bit \
  --namespace logging \
  -f fluent-bit.yaml
```

## 9. Verify Fluent Bit Installation

Check all pods:

```bash
kubectl get pods -n logging
```

Check the DaemonSet:

```bash
kubectl get daemonset -n logging
```

Check Fluent Bit logs:

```bash
kubectl logs -n logging -l app.kubernetes.io/name=fluent-bit --tail=100
```

Fluent Bit runs as a DaemonSet, meaning one Fluent Bit pod runs on every eligible Kubernetes node.

## 10. Deploy Sample Applications

Now I will deploy sample applications to generate logs.

Create a separate namespace:

```bash
kubectl create namespace dev
```

Navigate to the Day 4 application manifests in the cloned repository:

```bash
cd ../day-4
```

Apply the application manifests provided in the instructor's repository:

```bash
kubectl apply -f .
```

Check the pods:

```bash
kubectl get pods -n dev
```

Wait until the application pods are Running.

## 11. Generate and Check Application Logs

Check the application logs:

```bash
kubectl logs -n dev <POD-NAME>
```

Check all application pods:

```bash
kubectl get pods -n dev
```

Check Fluent Bit logs:

```bash
kubectl logs -n logging -l app.kubernetes.io/name=fluent-bit --tail=200
```

Verify that Fluent Bit is detecting application workloads and forwarding their logs to Elasticsearch.

## 12. Visualize Logs in Kibana

Open the Kibana LoadBalancer address in my browser.

Login using:

```text
Username: elastic
Password: <ELASTICSEARCH-PASSWORD>
```

Follow these steps:

1. Open Kibana.
2. Navigate to Discover or Data Views, depending on the Kibana version.
3. Create a Data View.
4. Enter the Elasticsearch index pattern used by Fluent Bit, such as `logstash-*` if that is the configured index.
5. Select the timestamp field, if prompted.
6. Save the Data View.
7. Open Discover.

I should now be able to see logs collected from my Kubernetes applications.

## 13. Filter Logs Using Kibana Query Language (KQL)

I can filter logs based on namespace or other available fields.

Example:

```text
kubernetes.namespace_name : "dev"
```

To search for error messages:

```text
log : "error"
```

The exact field names depend on the Fluent Bit configuration and indexed documents.

## 14. Verify the Complete EFK Stack

Check all components:

```bash
kubectl get pods -n logging
```

Check services:

```bash
kubectl get svc -n logging
```

Check DaemonSets:

```bash
kubectl get daemonset -n logging
```

Check Elasticsearch storage:

```bash
kubectl get pvc -n logging
```

Check application pods:

```bash
kubectl get pods -n dev
```

**Expected result:**

- Elasticsearch pods are Running.
- Kibana is Running and accessible.
- Fluent Bit pods are Running on the Kubernetes nodes.
- Application logs are collected by Fluent Bit.
- Logs are stored in Elasticsearch.
- Logs are visible and searchable in Kibana.

---

## Practical Completed

I have successfully deployed the EFK stack on my Kubernetes cluster.

**Final Log Flow:**

Application Pods → Fluent Bit → Elasticsearch → Kibana

I can now collect, store, search and visualize logs from multiple Kubernetes applications in one centralized logging system.
