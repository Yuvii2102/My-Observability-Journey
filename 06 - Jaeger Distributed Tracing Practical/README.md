# Day 6 – Jaeger Distributed Tracing Practical

## 1. Objective

In this practical, I implemented Distributed Tracing using Jaeger in an AWS EKS cluster.

I configured Jaeger to use Elasticsearch as its storage backend and deployed two instrumented microservices, Service A and Service B.

The main objective was to understand how requests travel between services and how Jaeger helps us visualise and troubleshoot those requests.

## 2. Tools and Technologies Used

- AWS EC2
- AWS EKS
- Kubernetes
- eksctl
- kubectl
- Helm
- Elasticsearch
- Jaeger
- OpenTelemetry
- Docker

## 3. Practical Flow

1. Prepare the new EC2 instance.
2. Create the EKS cluster.
3. Configure the EBS CSI Driver.
4. Install Elasticsearch.
5. Retrieve Elasticsearch credentials.
6. Configure Jaeger.
7. Install Jaeger using Helm.
8. Access the Jaeger UI.
9. Deploy Service A and Service B.
10. Generate requests and view traces.
11. Clean up everything.

---

# Phase 1 – Prepare the EC2 Instance

First, I connected to my Ubuntu EC2 instance and installed the required tools.

### Step 1: Update the system

```bash
sudo apt update
```

Install the required packages:

```bash
sudo apt install unzip curl wget git jq -y
```

**What I did:** Updated the package list and installed the basic utilities required for this practical.

### Step 2: Install AWS CLI

Download AWS CLI:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

Extract the downloaded file:

```bash
unzip awscliv2.zip
```

Install AWS CLI:

```bash
sudo ./aws/install
```

Verify the installation:

```bash
aws --version
```

Configure AWS credentials:

```bash
aws configure
```

I entered my AWS Access Key, Secret Access Key, default region and output format.

**What I did:** Installed AWS CLI and configured it to communicate with my AWS account.

### Step 3: Install kubectl

Download kubectl:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Install kubectl:

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Verify:

```bash
kubectl version --client
```

**What I did:** Installed kubectl to manage Kubernetes resources and interact with my EKS cluster.

### Step 4: Install eksctl

Download and extract eksctl:

```bash
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
```

Move it to the system path:

```bash
sudo mv /tmp/eksctl /usr/local/bin
```

Verify:

```bash
eksctl version
```

**What I did:** Installed eksctl to create and manage my Amazon EKS cluster.

### Step 5: Install Helm

Install Helm:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verify:

```bash
helm version
```

**What I did:** Installed Helm to deploy applications and Kubernetes components using Helm charts.

---

# Phase 2 – Create the EKS Cluster

In this phase, I created an EKS cluster named `observability`.

### Step 6: Create EKS cluster

```bash
eksctl create cluster --name=observability \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup
```

**What I did:**

- Created an EKS cluster named `observability`.
- Used the `us-east-1` region.
- Selected two availability zones.
- Created the cluster without a node group initially.

### Step 7: Associate OIDC provider

```bash
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster observability \
    --approve
```

**What I did:** Associated an IAM OIDC provider with the EKS cluster. This allows Kubernetes service accounts to use IAM roles.

### Step 8: Create node group

```bash
eksctl create nodegroup --cluster=observability \
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

**What I did:**

- Created a managed node group.
- Used `t3.medium` EC2 instances.
- Configured a minimum of 2 and maximum of 3 nodes.
- Configured private networking for the worker nodes.

### Step 9: Update kubeconfig

```bash
aws eks update-kubeconfig --name observability
```

Verify the nodes:

```bash
kubectl get nodes
```

**Expected outcome:** The EKS cluster should be accessible, and the worker nodes should appear in the output.

---

# Phase 3 – Configure EBS CSI Driver

The EBS CSI Driver allows Kubernetes to provision and manage Amazon EBS volumes for persistent storage.

### Step 10: Create IAM service account

```bash
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster observability \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEKS_EBS_CSI_DriverPolicy \
    --approve
```

**What I did:** Created an IAM service account and attached the required EBS CSI Driver policy.

### Step 11: Retrieve IAM role ARN

```bash
ARN=$(aws iam get-role --role-name AmazonEKS_EBS_CSI_DriverRole --query 'Role.Arn' --output text)
```

Display the ARN:

```bash
echo $ARN
```

**What I did:** Retrieved the IAM role ARN and stored it in the `ARN` variable.

### Step 12: Install EBS CSI Driver

```bash
eksctl create addon --cluster observability --name aws-ebs-csi-driver --version latest \
    --service-account-role-arn $ARN --force
```

**What I did:** Installed the AWS EBS CSI Driver add-on and associated it with the IAM role.

---

# Phase 4 – Install Elasticsearch

Jaeger needs a storage backend to store collected trace data. In this practical, I used Elasticsearch.

### Step 13: Create logging namespace

```bash
kubectl create namespace logging
```

**What I did:** Created a separate namespace named `logging` for Elasticsearch.

### Step 14: Add Elasticsearch Helm repository

```bash
helm repo add elastic https://helm.elastic.co
```

**What I did:** Added the official Elastic Helm repository.

### Step 15: Install Elasticsearch

```bash
helm install elasticsearch \
 --set replicas=1 \
 --set volumeClaimTemplate.storageClassName=gp2 \
 --set persistence.labels.enabled=true elastic/elasticsearch -n logging
```

**What I did:**

- Installed Elasticsearch using Helm.
- Configured one replica.
- Used the `gp2` storage class for persistent storage.
- Deployed Elasticsearch in the `logging` namespace.

### Step 16: Retrieve Elasticsearch username

```bash
kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.username}' | base64 -d
```

**What I did:** Retrieved the Elasticsearch username stored in a Kubernetes Secret.

### Step 17: Retrieve Elasticsearch password

```bash
kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
```

**What I did:** Retrieved the Elasticsearch password.

**Important:** I saved the password securely because it is required in the Jaeger configuration.

---

# Phase 5 – Configure Jaeger

In this phase, I downloaded the practical repository and configured Jaeger to communicate securely with Elasticsearch.

### Step 18: Download the repository

```bash
git clone https://github.com/iam-veeramalla/observability-zero-to-hero.git
```

Navigate to the Day 6 directory:

```bash
cd observability-zero-to-hero/day-6
```

Check the files:

```bash
ls
```

The directory should contain:

- `jaeger-values.yaml`
- `readme.md`

### Step 19: Extract Elasticsearch CA certificate

```bash
kubectl get secret elasticsearch-master-certs -n logging -o jsonpath='{.data.ca\.crt}' | base64 --decode > ca-cert.pem
```

**What I did:**

- Retrieved the Elasticsearch CA certificate.
- Decoded the certificate from Base64.
- Saved it as `ca-cert.pem`.

This certificate is required for secure communication between Jaeger and Elasticsearch.

### Step 20: Create tracing namespace

```bash
kubectl create ns tracing
```

**What I did:** Created a separate namespace for Jaeger resources.

### Step 21: Create ConfigMap

```bash
kubectl create configmap jaeger-tls --from-file=ca-cert.pem -n tracing
```

**What I did:** Stored the CA certificate in a ConfigMap named `jaeger-tls`.

### Step 22: Create Kubernetes Secret

```bash
kubectl create secret generic es-tls-secret --from-file=ca-cert.pem -n tracing
```

**What I did:** Created a Kubernetes Secret named `es-tls-secret` containing the certificate file.

### Step 23: Add Jaeger Helm repository

```bash
helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
```

Update Helm repositories:

```bash
helm repo update
```

**What I did:** Added and updated the Jaeger Helm chart repository.

### Step 24: Update Jaeger values file

Open the configuration file:

```bash
vim jaeger-values.yaml
```

Find the Elasticsearch storage configuration:

```yaml
storage:
  type: elasticsearch
  elasticsearch:
    host: elasticsearch-master.logging.svc
    port: 9200
    scheme: https
    user: elastic
    password: YOUR_ELASTICSEARCH_PASSWORD
```

Replace `YOUR_ELASTICSEARCH_PASSWORD` with the password retrieved in Step 17.

Save and exit:

```text
Esc
:wq
```

**What I did:** Updated the Elasticsearch password so Jaeger can authenticate with Elasticsearch.

The supplied values file also configures TLS and the CA certificate mount.

**Important:** The values file must be compatible with the Jaeger Helm chart version used in the teacher's practical. I should not mix this legacy configuration with the current Jaeger v2 chart.

### Step 25: Install Jaeger

```bash
helm install jaeger jaegertracing/jaeger -n tracing --values jaeger-values.yaml
```

**What I did:** Installed Jaeger using Helm and supplied my custom configuration file.

### Step 26: Verify Jaeger resources

Check the pods:

```bash
kubectl get pods -n tracing
```

Check the services:

```bash
kubectl get svc -n tracing
```

**What I did:** Verified that Jaeger components and services were created in the `tracing` namespace.

---

# Phase 6 – Access Jaeger UI

### Step 27: Port-forward Jaeger Query

```bash
kubectl port-forward svc/jaeger-query 8080:80 -n tracing
```

**What I did:** Forwarded local port `8080` to port `80` of the Jaeger Query service.

I kept this command running while accessing the dashboard.

Open the Jaeger UI:

```text
http://localhost:8080
```

If I am accessing Jaeger from my local computer, I need an SSH tunnel to forward the EC2 port to my computer.

---

# Phase 7 – Deploy Instrumented Applications

For this practical, I used the Day 4 applications because they already contain OpenTelemetry instrumentation.

### Step 28: Navigate to Day 4

In another terminal:

```bash
cd ~/observability-zero-to-hero/day-4
```

### Step 29: Create dev namespace

```bash
kubectl create ns dev
```

**What I did:** Created a namespace named `dev` for the application deployments.

### Step 30: Deploy application manifests

Before deploying, I checked the image names in the deployment YAML files.

The image names must point to images that are available to the Kubernetes cluster.

Deploy the application:

```bash
kubectl apply -k kubernetes-manifest/
```

**What I did:** Applied the Kubernetes manifests using Kustomize to deploy Service A and Service B.

### Step 31: Verify deployments

Check the pods:

```bash
kubectl get pods -n dev
```

Check the services:

```bash
kubectl get svc -n dev
```

**What I did:** Verified that Service A and Service B were deployed.

### Step 32: Get Service A LoadBalancer address

```bash
kubectl get svc -n dev
```

I copied the external address of Service A.

### Step 33: Test the healthy endpoint

```bash
curl http://<LOAD-BALANCER-ADDRESS>/healthy
```

**What I did:** Sent a request to the healthy endpoint to check whether Service A was responding.

### Step 34: Call Service B through Service A

```bash
curl http://<LOAD-BALANCER-ADDRESS>/call-service-b
```

**What I did:** Sent a request to Service A that triggers communication with Service B.

This request helps generate a distributed trace across both services.

---

# Phase 8 – View Traces in Jaeger

Open the Jaeger UI:

```text
http://localhost:8080
```

Follow these steps:

1. Select Service A from the Service dropdown.
2. Select an operation.
3. Click **Find Traces**.
4. Open a trace to inspect its details.
5. Check the spans and execution duration.
6. Select Service B.
7. Search for traces again.
8. Inspect the communication between both services.

### Expected Outcome

I should be able to see:

- Service A and Service B in Jaeger.
- Traces generated by requests to the application.
- Multiple spans representing operations across services.
- The request flow between Service A and Service B.
- Execution time for individual operations.

---

# Phase 9 – Cleanup

After completing the practical, I can remove the resources using the cleanup commands.

### Step 1: Uninstall Jaeger

```bash
helm uninstall jaeger -n tracing
```

### Step 2: Uninstall Elasticsearch

```bash
helm uninstall elasticsearch -n logging
```

### Step 3: Uninstall Monitoring

```bash
helm uninstall monitoring -n monitoring
```

### Step 4: Navigate to Day 4

```bash
cd ../day-4
```

### Step 5: Delete Kubernetes application manifests

```bash
kubectl delete -k kubernetes-manifest/
```

### Step 6: Delete alerts and monitoring manifests

```bash
kubectl delete -k alerts-alertmanager-servicemonitor-manifest/
```

### Step 7: Delete EKS cluster

```bash
eksctl delete cluster --name observability
```

**Important:** I should verify that any remaining EBS volumes and Load Balancers are removed to avoid unnecessary AWS charges.

---

# 10. What I Learned

Through this practical, I learned:

1. How distributed tracing works in microservices.
2. How Jaeger collects and visualises trace data.
3. How to configure Elasticsearch as Jaeger's storage backend.
4. How to configure secure communication using TLS certificates.
5. How to deploy Jaeger using Helm.
6. How to access the Jaeger UI using port forwarding.
7. How to deploy instrumented applications on EKS.
8. How to generate traces by sending requests between services.
9. How to analyse spans and understand request execution time.

## 11. Final Outcome

I successfully learned the workflow for implementing distributed tracing using Jaeger and Elasticsearch on AWS EKS.

I understood how requests travel between multiple microservices and how Jaeger helps visualise their execution through traces and spans.

**Day 6 – Distributed Tracing with Jaeger.**
