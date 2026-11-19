# Day 4 – Custom Metrics Instrumentation and Alerting on Amazon EKS

## 1. Practical Overview

In this practical, I implemented custom application metrics and configured alerting using Prometheus and Alertmanager on my Amazon EKS cluster.

I deployed a Node.js microservices application consisting of Service A and Service B. I used Prometheus to collect application metrics, Grafana to visualize them, and Alertmanager to send email notifications when an alert condition was met.

I also intentionally crashed the application to test whether Kubernetes could restart the container and whether Prometheus and Alertmanager could detect the restarts and send an email notification.

### Tools and Technologies Used

- Amazon EKS
- Kubernetes
- Docker
- Prometheus
- Grafana
- Alertmanager
- Prometheus Operator
- ServiceMonitor
- PrometheusRule
- Kustomize
- Node.js
- Gmail SMTP

### What I Implemented

1. Deployed a Node.js microservices application on EKS.
2. Exposed application metrics through the `/metrics` endpoint.
3. Configured a ServiceMonitor for Prometheus service discovery.
4. Verified custom metrics in Grafana.
5. Created Prometheus alert rules for high CPU usage and pod restarts.
6. Configured Alertmanager to send email notifications through Gmail.
7. Intentionally crashed the application to test the PodRestart alert.
8. Verified the alert in Prometheus and received the email notification in Gmail.

---

## 2. Prerequisites

Before starting this practical, I already had the following components running from my previous Observability practicals:

- An Amazon EKS cluster.
- Two healthy EKS worker nodes.
- Prometheus installed through the Prometheus Community kube-prometheus-stack Helm chart.
- Grafana installed.
- Alertmanager installed.
- Node Exporter and Kube State Metrics running.

I used the following environment:

| Component | Name |
|---|---|
| AWS Region | `us-east-1` |
| EKS Cluster | `observability` |
| Monitoring Namespace | `monitoring` |
| Application Namespace | `dev` |
| Repository | `observability-zero-to-hero` |
| Practical Folder | `day-4` |

**Important:** I did not reinstall Prometheus, Grafana or Alertmanager because they were already installed from the previous practical.

### Step 1: Verify the EKS cluster

I checked whether my EKS cluster was available.

```bash
eksctl get cluster --region us-east-1
```

### Step 2: Verify worker nodes

```bash
kubectl get nodes
```

I confirmed that both worker nodes were in the `Ready` state.

### Step 3: Verify monitoring components

```bash
kubectl get pods -n monitoring
```

I checked that Prometheus, Grafana, Alertmanager, Node Exporter and Kube State Metrics were running.

---

## 3. Clone and Navigate to the Day 4 Repository

I followed the Day 4 practical from Abhishek Veeramalla's Observability Zero to Hero repository.

**Original GitHub repository:**

https://github.com/iam-veeramalla/observability-zero-to-hero/tree/main/day-4

Since I had already cloned the repository in my EC2 instance, I navigated to the existing repository.

```bash
cd ~/observability-zero-to-hero
```

I moved into the Day 4 folder.

```bash
cd day-4
```

I checked the available files and folders.

```bash
ls -la
```

My Day 4 folder contained:

```text
day-4/
│
├── alerts-alertmanager-servicemonitor-manifest/
│
├── application/
│
├── images/
│
├── kubernetes-manifest/
│
├── readme.md
│
└── test.sh
```

### Purpose of Each Folder

| Folder/File | Purpose |
|---|---|
| `application` | Contains the Node.js application source code |
| `kubernetes-manifest` | Contains Kubernetes Deployment and Service YAML files |
| `alerts-alertmanager-servicemonitor-manifest` | Contains alert rules, ServiceMonitor, Alertmanager configuration and email Secret |
| `images` | Contains practical-related images |
| `readme.md` | Contains the practical instructions |
| `test.sh` | Contains the provided test script |

---

## 4. Understand the Application and Custom Metrics

The application contains two services:

- Service A
- Service B

Service A is exposed through a LoadBalancer so that I can access it externally.

Service B is exposed internally through a ClusterIP Service. Service A communicates with Service B using Kubernetes service discovery.

The application uses the Node.js `prom-client` library to expose custom metrics.

### Custom Metrics Implemented

The application exposes four custom metrics.

| Metric | Type | Purpose |
|---|---|---|
| `http_requests_total` | Counter | Tracks total HTTP requests |
| `http_request_duration_seconds` | Histogram | Measures HTTP request duration |
| `http_request_duration_summary_seconds` | Summary | Provides request duration statistics |
| `node_gauge_example` | Gauge | Tracks an example asynchronous task duration |

### Application Endpoints

| Endpoint | Purpose |
|---|---|
| `/` | Returns the application running status |
| `/healthy` | Returns the health status |
| `/serverError` | Simulates a 500 Internal Server Error |
| `/notFound` | Simulates a 404 Not Found error |
| `/logs` | Generates application logs |
| `/crash` | Simulates an application crash |
| `/example` | Tracks asynchronous task duration |
| `/metrics` | Exposes Prometheus metrics |
| `/call-service-b` | Calls Service B from Service A |

**Source:** The application details and metric names are documented in the original Day 4 README and application source code.

---

## 5. Deploy the Node.js Microservices Application

### Step 1: Navigate to Kubernetes manifests

```bash
cd ~/observability-zero-to-hero/day-4/kubernetes-manifest
```

I listed the files.

```bash
ls -la
```

The folder contained:

```text
deployment-svc-a.yml
deployment-svc-b.yml
kustomization.yml
service-svc-a.yml
service-svc-b.yml
```

### Step 2: Verify the Docker images

The practical uses the following prebuilt Docker images:

```text
abhishekf5/demoservice-a:v
abhishekf5/demoservice-b:v
```

I verified that the Service B image was available:

```bash
docker manifest inspect abhishekf5/demoservice-b:v
```

**Important correction from my practical:** The original Service B Deployment manifest referenced the Service A image. I corrected the image in `deployment-svc-b.yml` to:

```yaml
image: abhishekf5/demoservice-b:v
```

This ensures that Service B uses its correct Docker image.

### Step 3: Create the application namespace

I created a separate namespace for the application.

```bash
kubectl create ns dev
```

Expected output:

```text
namespace/dev created
```

If the namespace already exists when repeating the practical, I can check it using:

```bash
kubectl get namespace dev
```

I do not need to create it again if it already exists.

### Step 4: Apply Kubernetes manifests using Kustomize

I moved back to the Day 4 folder.

```bash
cd ~/observability-zero-to-hero/day-4
```

I deployed the application using Kustomize.

```bash
kubectl apply -k kubernetes-manifest/
```

Expected output:

```text
service/a-service created
service/b-service created
deployment.apps/service-a-deployment created
deployment.apps/service-b-deployment created
```

This command creates the application Deployments and Services using the resources listed in `kustomization.yml`.

### Step 5: Verify application pods

```bash
kubectl get pods -n dev
```

I checked that both Service A and Service B pods were running.

Expected status:

```text
NAME                                    READY   STATUS
service-a-deployment-xxxxxxxxxx         1/1     Running
service-b-deployment-xxxxxxxxxx         1/1     Running
```

The actual pod names are generated by Kubernetes, so they may be different.

### Step 6: Verify Kubernetes Services

```bash
kubectl get svc -n dev
```

I observed two services:

| Service | Type | Purpose |
|---|---|---|
| `a-service` | LoadBalancer | External access to Service A |
| `b-service` | ClusterIP | Internal communication with Service B |

### Step 7: Get the LoadBalancer DNS

I retrieved the external DNS address using:

```bash
kubectl get svc a-service -n dev
```

To retrieve only the DNS address:

```bash
kubectl get svc a-service -n dev -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

I stored the DNS address in a variable so that I could reuse it in the following commands.

```bash
APP_DNS=$(kubectl get svc a-service -n dev -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
```

I displayed the DNS address:

```bash
echo "$APP_DNS"
```

**Note:** The LoadBalancer may take a few minutes to become available.

---

## 6. Test the Application Endpoints

Before configuring monitoring, I tested the application through its LoadBalancer DNS.

### 1. Root endpoint

```bash
curl "http://$APP_DNS/"
```

Expected response:

```json
{"status":"🏃- Running"}
```

This confirms that the application is running.

### 2. Health endpoint

```bash
curl "http://$APP_DNS/healthy"
```

Expected response indicates that the application is healthy.

### 3. Server error endpoint

```bash
curl "http://$APP_DNS/serverError"
```

Expected response:

```json
{"error":" Internal server error","statusCode":500}
```

This endpoint generates an HTTP 500 response.

### 4. Not Found endpoint

```bash
curl "http://$APP_DNS/notFound"
```

Expected response:

```json
{"error":"Not Found","statusCode":"404"}
```

### 5. Logs endpoint

```bash
curl "http://$APP_DNS/logs"
```

Expected response:

```json
{"objective":"To generate logs"}
```

### 6. Example endpoint

```bash
curl "http://$APP_DNS/example"
```

Expected response:

```text
Async task completed
```

### 7. Service-to-service communication

```bash
curl "http://$APP_DNS/call-service-b"
```

Expected response:

```html
<h1 style="font-size: 100px">Service B says: Hello from Service B!<h1>
```

This confirms that Service A can communicate with Service B internally.

### 8. Metrics endpoint

```bash
curl "http://$APP_DNS/metrics"
```

This endpoint exposes the custom metrics in Prometheus-compatible format.

I verified that the output contained the following metric names:

```text
http_requests_total

http_request_duration_seconds

http_request_duration_summary_seconds

node_gauge_example
```

---

## 7. Configure Prometheus Service Discovery Using ServiceMonitor

At this stage, the application was generating custom metrics, but Prometheus needed to know which Kubernetes Service to monitor.

For this, I used a Kubernetes custom resource called `ServiceMonitor`.

### Step 1: Navigate to the configuration folder

```bash
cd ~/observability-zero-to-hero/day-4/alerts-alertmanager-servicemonitor-manifest
```

I listed the files.

```bash
ls -la
```

The folder contained:

```text
alertmangerconfig.yml
alerts.yml
email-secrets.yml
kustomization.yml
serviceMonitor.yml
```

### Step 2: Inspect the ServiceMonitor

```bash
cat serviceMonitor.yml
```

The configuration:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app: a-service-service-monitor
    release: monitoring
  name: a-service-service-monitor
  namespace: monitoring
spec:
  jobLabel: job
  endpoints:
    - interval: 2s
      port: a-service-port
      path: /metrics
  selector:
    matchLabels:
      app: a-service
  namespaceSelector:
    matchNames:
      - dev
```

### Understanding the Configuration

| Field | Purpose |
|---|---|
| `kind: ServiceMonitor` | Creates a ServiceMonitor resource |
| `namespace: monitoring` | Creates the ServiceMonitor in the monitoring namespace |
| `release: monitoring` | Matches the Prometheus Operator release label |
| `interval: 2s` | Configures the scrape interval to 2 seconds |
| `port: a-service-port` | Selects the named Service port |
| `path: /metrics` | Tells Prometheus to collect metrics from `/metrics` |
| `matchLabels` | Selects the Service with the label `app: a-service` |
| `matchNames: dev` | Looks for the application Service in the `dev` namespace |

The ServiceMonitor selects the `a-service` Service and tells Prometheus where to collect metrics.

---

## 8. Configure Prometheus Alert Rules

I created two custom alert rules:

1. HighCpuUsage
2. PodRestart

These rules are defined in `alerts.yml`.

### Step 1: Inspect the alert rules

```bash
cat alerts.yml
```

The configuration:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: custom-alert-rules
  namespace: monitoring
  labels:
    release: monitoring
spec:
  groups:
  - name: custom.rules
    rules:
    - alert: HighCpuUsage
      expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 50
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High CPU usage on instance {{ $labels.instance }}"
        description: "CPU usage is above 50% (current value: {{ $value }}%)"

    - alert: PodRestart
      expr: kube_pod_container_status_restarts_total > 2
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: "Pod restart detected in namespace {{ $labels.namespace }}"
        description: "Pod {{ $labels.pod }} in namespace {{ $labels.namespace }} has restarted {{ $value }} times"
```

### Alert 1: HighCpuUsage

The PromQL expression:

```promql
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 50
```

This calculates CPU usage based on idle CPU time.

The alert triggers when CPU usage stays above 50% for 5 minutes.

| Field | Value |
|---|---|
| Alert name | `HighCpuUsage` |
| Condition | CPU usage above 50% |
| Duration | 5 minutes |
| Severity | warning |

### Alert 2: PodRestart

The PromQL expression:

```promql
kube_pod_container_status_restarts_total > 2
```

This checks the total number of container restarts.

The alert triggers when a container has restarted more than two times.

| Field | Value |
|---|---|
| Alert name | `PodRestart` |
| Condition | More than 2 restarts |
| Duration | 0 minutes |
| Severity | critical |

`for: 0m` means the alert does not need to remain true for an additional waiting period before firing.

---

## 9. Configure Alertmanager Email Notifications

I configured Alertmanager to send email notifications when the custom alert rules fire.

I used Gmail SMTP for email delivery.

### Step 1: Inspect Alertmanager configuration

```bash
cat alertmangerconfig.yml
```

The configuration:

```yaml
apiVersion: monitoring.coreos.com/v1alpha1
kind: AlertmanagerConfig
metadata:
  name: main-rules-alert-config
  namespace: monitoring
  labels:
    release: monitoring
spec:
  route:
    repeatInterval: 30m
    receiver: 'null'
    routes:
    - matchers:
      - name: alertname
        value: HighCpuUsage
      receiver: 'send-email'

    - matchers:
      - name: alertname
        value: PodRestart
      receiver: 'send-email'
      repeatInterval: 5m

  receivers:
  - name: 'send-email'
    emailConfigs:
    - to: YOUR_EMAIL_ID
      from: YOUR_EMAIL_ID
      sendResolved: false
      smarthost: smtp.gmail.com:587
      authUsername: YOUR_EMAIL_ID
      authIdentity: YOUR_EMAIL_ID
      authPassword:
        name: mail-pass
        key: gmail-pass

  - name: 'null'
```

### Understanding the Configuration

- `receiver: send-email` sends matching alerts to the email receiver.
- `HighCpuUsage` is routed to `send-email`.
- `PodRestart` is routed to `send-email`.
- `smtp.gmail.com:587` is the Gmail SMTP server.
- `to` is the recipient email address.
- `from` is the sender email address.
- `authUsername` and `authIdentity` use the Gmail address.
- `authPassword` reads the password from the Kubernetes Secret named `mail-pass`.
- `sendResolved: false` means resolved notifications are not sent.

I replaced all four `YOUR_EMAIL_ID` placeholders with my Gmail address.

```yaml
to: YOUR_EMAIL_ID
from: YOUR_EMAIL_ID
authUsername: YOUR_EMAIL_ID
authIdentity: YOUR_EMAIL_ID
```

I kept the remaining configuration unchanged.

---

## 10. Create the Gmail App Password Secret

Alertmanager requires authentication to send email through Gmail SMTP.

I used a Gmail App Password instead of my normal Gmail account password.

### Step 1: Generate a Gmail App Password

1. Open my Google Account.
2. Go to Manage Google Account.
3. Enable 2-Step Verification if it is not already enabled.
4. Search for App passwords.
5. Create a new App Password.
6. Name it `alertmanager`.
7. Copy the generated password.

### Step 2: Inspect the Secret manifest

```bash
cat email-secrets.yml
```

The configuration:

```yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: mail-pass
  namespace: monitoring
  labels:
    release: monitoring
data:
  gmail-pass: YOUR_BASE64_ENCODED_PASSWORD
```

The Secret name is `mail-pass`, and the key is `gmail-pass`.

### Step 3: Convert the App Password into Base64

I used the following command on my Ubuntu EC2 instance:

```bash
echo -n 'YOUR_GMAIL_APP_PASSWORD' | base64 -w 0
```

I replaced the placeholder with my generated Gmail App Password.

The command returns a Base64-encoded value.

I copied the output.

**Important:** Base64 is encoding, not encryption. I must keep this value private.

### Step 4: Update the Secret YAML file

I opened the file:

```bash
vim email-secrets.yml
```

I replaced the placeholder with my Base64 output:

```yaml
data:
  gmail-pass: YOUR_BASE64_ENCODED_PASSWORD
```

I saved the file using:

```text
Esc
:wq
Enter
```

I did not share my App Password or Base64 value in chat.

I also avoided committing real credentials to a public GitHub repository.

---

## 11. Understand the Kustomization File

Before applying the configurations, I inspected the Kustomization file.

```bash
cat kustomization.yml
```

The configuration:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: monitoring
resources:
  - alerts.yml
  - email-secrets.yml
  - alertmangerconfig.yml
  - serviceMonitor.yml
```

This file tells Kustomize which Kubernetes resources to apply.

| File | Purpose |
|---|---|
| `alerts.yml` | Creates Prometheus alert rules |
| `email-secrets.yml` | Stores the Gmail App Password |
| `alertmangerconfig.yml` | Configures Alertmanager email notifications |
| `serviceMonitor.yml` | Configures Prometheus service discovery |

**Important:** These are the exact filenames in my checked-out practical folder. The repository README may use slightly different filenames in its written instructions, so I used the filenames actually present in my local folder.

---

## 12. Apply All Alerting and Monitoring Configurations

After updating the Gmail Secret and email configuration, I applied all the manifests together.

I was already inside:

```text
~/observability-zero-to-hero/day-4/alerts-alertmanager-servicemonitor-manifest
```

I executed:

```bash
kubectl apply -k .
```

Expected output:

```text
secret/mail-pass created
prometheusrule.monitoring.coreos.com/custom-alert-rules created
servicemonitor.monitoring.coreos.com/a-service-service-monitor created
alertmanagerconfig.monitoring.coreos.com/main-rules-alert-config created
```

This applied all four resources successfully.

---

## 13. Verify the Created Resources

I verified whether the ServiceMonitor, PrometheusRule and AlertmanagerConfig resources were created.

```bash
kubectl get servicemonitor,prometheusrule,alertmanagerconfig -n monitoring
```

I checked for:

```text
a-service-service-monitor
custom-alert-rules
main-rules-alert-config
```

I also verified the Secret without displaying its contents:

```bash
kubectl get secret mail-pass -n monitoring
```

The Secret should be present in the `monitoring` namespace.

---

## 14. Access Grafana Through Port Forwarding

Grafana was exposed as a ClusterIP Service.

I checked the monitoring services:

```bash
kubectl get svc -n monitoring
```

I found:

```text
monitoring-grafana   ClusterIP   10.x.x.x   <none>   80/TCP
```

Since ClusterIP is accessible internally within the cluster, I used Kubernetes port forwarding to access Grafana through my browser.

### Step 1: Start Grafana port forwarding

```bash
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring --address 0.0.0.0
```

Expected output:

```text
Forwarding from 0.0.0.0:3000 -> 80
```

I kept this terminal running.

### Step 2: Configure the EC2 Security Group

I added an inbound rule for Grafana:

| Type | Protocol | Port | Source |
|---|---|---|---|
| Custom TCP | TCP | 3000 | My public IP/32 |

I used my own IP address instead of opening the port to everyone.

### Step 3: Open Grafana

I opened the following address in my browser:

```text
http://EC2_PUBLIC_IP:3000
```

I replaced `EC2_PUBLIC_IP` with my EC2 instance's public IPv4 address.

### Step 4: Retrieve Grafana login credentials

I retrieved the username:

```bash
kubectl get secret -n monitoring monitoring-grafana -o jsonpath="{.data.admin-user}" | base64 -d; echo
```

I retrieved the password:

```bash
kubectl get secret -n monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d; echo
```

I used the returned credentials to log in.

---

## 15. Verify Prometheus Scraping in Grafana

After logging in to Grafana, I opened:

```text
Explore → Prometheus
```

I selected the Prometheus data source and switched to Code mode.

### Step 1: Check whether the application target is UP

I executed:

```promql
up{namespace="dev"}
```

Expected value:

```text
1
```

My result was `1`.

This confirmed that Prometheus was successfully scraping the application.

I also verified the labels:

```text
namespace="dev"
service="a-service"
job="node-api"
```

### Understanding the Result

| Value | Meaning |
|---|---|
| `1` | Prometheus successfully scraped the target |
| `0` | The target scrape failed |

---

## 16. Verify the Four Custom Metrics in Grafana

I checked the four custom metrics one by one in Grafana Explore.

For each query, I entered the metric name in Code mode and clicked Run queries.

### 1. Verify HTTP Requests Counter

```promql
http_requests_total
```

This displayed the total number of HTTP requests received by the application.

I could see different labels for endpoints such as:

```text
path="/"
path="/healthy"
path="/call-service-b"
```

### 2. Verify HTTP Request Duration Histogram

```promql
http_request_duration_seconds
```

Grafana identified this as a histogram metric.

It returned the histogram buckets, count and sum series.

### 3. Verify HTTP Request Duration Summary

```promql
http_request_duration_summary_seconds
```

This displayed the summary metric series and quantile information.

### 4. Verify Node Gauge Example

```promql
node_gauge_example
```

This displayed the current gauge value.

### Verification Results

| Metric | Expected result |
|---|---|
| `http_requests_total` | Request counter data |
| `http_request_duration_seconds` | Histogram data |
| `http_request_duration_summary_seconds` | Summary data |
| `node_gauge_example` | Gauge value |

All four metrics were successfully visible in Grafana.

---

## 17. Verify Prometheus Alert Rules

I opened the Prometheus UI to verify whether Prometheus had loaded my custom alert rules.

### Step 1: Verify the Prometheus pod

```bash
kubectl get pods -n monitoring | grep prometheus
```

I identified the Prometheus pod:

```text
prometheus-monitoring-kube-prometheus-prometheus-0
```

### Step 2: Start Prometheus port forwarding

I opened another EC2 terminal and ran:

```bash
kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring --address 0.0.0.0
```

Expected output:

```text
Forwarding from 0.0.0.0:9090 -> 9090
```

I kept this terminal running.

### Step 3: Configure the EC2 Security Group

I added an inbound rule:

| Type | Protocol | Port | Source |
|---|---|---|---|
| Custom TCP | TCP | 9090 | My public IP/32 |

### Step 4: Open Prometheus

I opened:

```text
http://EC2_PUBLIC_IP:9090
```

### Step 5: Open the Rules page

In Prometheus:

1. Clicked **Status**.
2. Selected **Rules**.
3. Located the group named `custom.rules`.
4. Verified `HighCpuUsage`.
5. Verified `PodRestart`.

Both rules were visible and showed `OK`.

This confirmed that Prometheus had successfully loaded and evaluated the custom rules.

**Important:** `OK` means the rule is loaded and evaluated successfully. It does not mean that the alert is currently firing.

### Step 6: Understand why the Grafana ALERTS query showed no data

I initially tried:

```promql
ALERTS{alertname="HighCpuUsage"}
```

and:

```promql
ALERTS{alertname="PodRestart"}
```

Both returned no data because the alerts were inactive at that time.

This was not a rule-loading failure. I verified the actual rule loading through the Prometheus Rules page.

---

## 18. Test the PodRestart Alert by Intentionally Crashing the Application

Now I tested whether Prometheus and Alertmanager could detect container restarts.

The application has a special endpoint:

```text
/crash
```

Calling this endpoint intentionally crashes the Service A application process.

Kubernetes then detects the container failure and restarts it.

### Step 1: Trigger the first crash

I used the Service A LoadBalancer DNS:

```bash
curl "http://$APP_DNS/crash"
```

The following response may appear:

```text
curl: (52) Empty reply from server
```

This is expected because the application process terminates before sending an HTTP response.

### Step 2: Check pod status

```bash
kubectl get pods -n dev
```

I checked the restart count of the Service A pod.

Example:

```text
NAME                                    READY   STATUS    RESTARTS
service-a-deployment-xxxxxxxxxx         1/1     Running   1
```

### Step 3: Wait for the application to recover

I waited for Kubernetes to restart the container.

I checked the pod again:

```bash
kubectl get pods -n dev
```

If the pod temporarily showed `Error` or `BackOff`, I waited for the restart process instead of continuously sending crash requests.

I could inspect the pod using:

```bash
kubectl describe pod POD_NAME -n dev
```

I checked the `State`, `Last State`, `Restart Count` and `Events` sections.

### Step 4: Trigger additional crashes

I called the crash endpoint again:

```bash
curl "http://$APP_DNS/crash"
```

After the container restarted, I checked:

```bash
kubectl get pods -n dev
```

I repeated this process until the Service A container restart count reached 3.

My final output showed:

```text
NAME                                    READY   STATUS    RESTARTS
service-a-deployment-xxxxxxxxxx         1/1     Running   3
service-b-deployment-xxxxxxxxxx         1/1     Running   0
```

The important condition was:

```promql
kube_pod_container_status_restarts_total > 2
```

Since the restart count reached 3, the alert condition became true.

---

## 19. Verify the PodRestart Alert in Prometheus

After the restart count reached 3, I opened the Prometheus Alerts page.

I visited:

```text
http://EC2_PUBLIC_IP:9090/alerts
```

I located the `custom.rules` group.

I observed:

```text
PodRestart
FIRING
```

The alert details showed:

```text
alertname="PodRestart"
severity="critical"
namespace="dev"
```

The alert value was:

```text
3
```

This confirmed that Prometheus detected the container restarts and triggered the PodRestart alert.

### What Happened Internally?

1. I called the `/crash` endpoint.
2. The Node.js application process crashed.
3. Kubernetes detected the container failure.
4. Kubernetes restarted the container.
5. The restart count increased.
6. Kube State Metrics exposed the container restart information.
7. Prometheus collected the metric.
8. The Prometheus alert rule evaluated the condition.
9. The `PodRestart` alert entered the `Firing` state.

---

## 20. Verify Gmail Notification from Alertmanager

Finally, I checked whether Alertmanager successfully delivered the notification.

I opened the Gmail account configured in `alertmangerconfig.yml`.

I checked the Inbox and located the alert email.

The email subject began with:

```text
[FIRING:1] dev (PodRestart ...)
```

The email contained the following information:

- Alert name: `PodRestart`
- Namespace: `dev`
- Severity: `critical`
- Pod name: Service A deployment pod
- Restart count: 3
- Alert state: Firing

The email description confirmed:

```text
Pod service-a-deployment-xxxxxxxxxx in namespace dev has restarted 3 times
```

This confirmed that Alertmanager successfully sent the notification through Gmail SMTP.

**Final result:** I successfully tested the complete alerting flow, from intentionally crashing the application to receiving the email notification.

---

## 21. Final Practical Verification

| Component | Verification | Result |
|---|---|---|
| EKS cluster | `eksctl get cluster --region us-east-1` | Successful |
| Worker nodes | `kubectl get nodes` | Ready |
| Application pods | `kubectl get pods -n dev` | Running |
| Application Services | `kubectl get svc -n dev` | Created |
| Service A LoadBalancer | External DNS | Available |
| Service B | Internal communication | Successful |
| Application metrics | `/metrics` endpoint | Available |
| ServiceMonitor | `kubectl get servicemonitor -n monitoring` | Created |
| PrometheusRule | `kubectl get prometheusrule -n monitoring` | Created |
| AlertmanagerConfig | `kubectl get alertmanagerconfig -n monitoring` | Created |
| Gmail Secret | `kubectl get secret mail-pass -n monitoring` | Created |
| Grafana | Custom metric queries | Successful |
| Prometheus | Custom alert rules | Loaded |
| PodRestart | Prometheus Alerts page | Firing |
| Gmail notification | Gmail Inbox | Received |

---

## 22. Troubleshooting Notes

These are the issues I encountered during the practical and how I handled them.

### Issue 1: Service B was using the wrong Docker image

**Problem:** The Service B Deployment manifest initially referenced the Service A image.

**Solution:** I corrected the image in `deployment-svc-b.yml`:

```yaml
image: abhishekf5/demoservice-b:v
```

I verified the image using:

```bash
docker manifest inspect abhishekf5/demoservice-b:v
```

### Issue 2: Grafana was not accessible directly

**Problem:** Grafana was exposed as a ClusterIP Service.

**Solution:** I used port forwarding:

```bash
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring --address 0.0.0.0
```

I accessed Grafana through:

```text
http://EC2_PUBLIC_IP:3000
```

### Issue 3: Prometheus was not accessible through the browser

**Problem:** The initial port-forward command listened on localhost, so I could not access it through the EC2 public IP.

**Solution:** I restarted port forwarding with:

```bash
kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring --address 0.0.0.0
```

I then accessed:

```text
http://EC2_PUBLIC_IP:9090
```

### Issue 4: Custom alert queries showed No data

**Problem:** The following queries initially showed no data:

```promql
ALERTS{alertname="HighCpuUsage"}
```

```promql
ALERTS{alertname="PodRestart"}
```

**Solution:** I opened the Prometheus Rules page and verified that both rules were loaded.

The alerts were simply inactive at that time. After triggering the restart condition, `PodRestart` entered the Firing state.

### Issue 5: Curl returned Empty reply from server

**Problem:** When I called the crash endpoint, I received:

```text
curl: (52) Empty reply from server
```

**Explanation:** The application process terminated before returning an HTTP response.

**Solution:** I checked the pod restart count:

```bash
kubectl get pods -n dev
```

I waited for Kubernetes to restart the container before sending another crash request.

---

## 23. Cleanup and Security

After completing the demonstration, I stopped the port-forwarding processes using:

```text
Ctrl + C
```

I stopped the Grafana and Prometheus port-forwarding terminals.

I also reviewed my EC2 Security Group and restricted the temporary ports to my own public IP address.

The ports used were:

| Port | Service |
|---|---|
| 3000 | Grafana |
| 9090 | Prometheus |

I avoided leaving these ports open to `0.0.0.0/0`.

If I want to remove the application after the demonstration, I can run:

```bash
kubectl delete namespace dev
```

**Warning:** This deletes the application resources in the `dev` namespace, including the LoadBalancer Service. I should only run it when I no longer need the application.

I kept the monitoring stack in the `monitoring` namespace because it was already being used for my previous practicals.

---

## 24. What I Learned from This Practical

Through this practical, I learned how custom application metrics are generated and collected by Prometheus.

I understood that creating custom metrics in an application is not enough. Prometheus also needs a ServiceMonitor to discover the application Service and collect metrics from its `/metrics` endpoint.

I learned how to:

- Work with Prometheus custom metrics.
- Understand Counter, Histogram, Summary and Gauge metrics.
- Configure ServiceMonitor for application service discovery.
- Create custom Prometheus alert rules.
- Configure Alertmanager email notifications.
- Store Gmail App Passwords in Kubernetes Secrets.
- Use Kustomize to apply multiple Kubernetes manifests.
- Monitor application metrics through Grafana.
- Intentionally crash an application and observe Kubernetes container recovery.
- Verify alert states in Prometheus.
- Receive email notifications through Alertmanager.

### End-to-End Flow

```text
Node.js Application
        |
        v
Custom Metrics Instrumentation
        |
        v
/metrics Endpoint
        |
        v
ServiceMonitor
        |
        v
Prometheus
        |
        v
Prometheus Alert Rules
        |
        v
Alertmanager
        |
        v
Gmail SMTP
        |
        v
Email Notification
```

### Application Crash and Alerting Flow

```text
/crash Endpoint
        |
        v
Application Container Crashes
        |
        v
Kubernetes Restarts Container
        |
        v
Kube State Metrics
        |
        v
Prometheus Detects Restarts
        |
        v
PodRestart Alert Fires
        |
        v
Alertmanager Sends Email
```

---

## 25. Exact Sources of the Commands and Configurations

### 1. Primary Source – Abhishek Veeramalla's GitHub Repository

I followed the Day 4 practical from Abhishek Veeramalla's Observability Zero to Hero repository.

**GitHub Repository:**

[Observability Zero to Hero – Day 4](https://github.com/iam-veeramalla/observability-zero-to-hero/tree/main/day-4)

I used this repository for the application source code, Kubernetes manifests, ServiceMonitor configuration, Prometheus alert rules, Alertmanager configuration and practical instructions.

### Important Repository Files

| Repository File | What I Used It For |
|---|---|
| [Day 4 README](https://github.com/iam-veeramalla/observability-zero-to-hero/blob/main/day-4/readme.md) | Main practical instructions and application endpoints |
| [Application source code](https://github.com/iam-veeramalla/observability-zero-to-hero/tree/main/day-4/application) | Node.js application and custom metrics |
| [Kubernetes manifests](https://github.com/iam-veeramalla/observability-zero-to-hero/tree/main/day-4/kubernetes-manifest) | Application Deployments and Services |
| [Alertmanager and ServiceMonitor manifests](https://github.com/iam-veeramalla/observability-zero-to-hero/tree/main/day-4/alerts-alertmanager-servicemonitor-manifest) | ServiceMonitor, PrometheusRule, AlertmanagerConfig and Secret |
| [Test script](https://github.com/iam-veeramalla/observability-zero-to-hero/blob/main/day-4/test.sh) | Automated application endpoint testing |

### 2. Teacher's Practical Demonstration

I followed Abhishek Veeramalla's Day 4 demonstration to understand and implement:

- Custom metrics instrumentation using Node.js and `prom-client`.
- Deployment of Service A and Service B.
- Prometheus ServiceMonitor configuration.
- Prometheus alert rules.
- Gmail App Password configuration.
- Alertmanager email notifications.
- Container crash testing through the `/crash` endpoint.

### 3. Source of the Main Commands

The following table explains where I got the commands used in this practical.

| Command | Source |
|---|---|
| `eksctl get cluster --region us-east-1` | EKS cluster verification used in my practical |
| `kubectl get nodes` | Kubernetes cluster verification |
| `kubectl create ns dev` | Day 4 README |
| `kubectl apply -k kubernetes-manifest/` | Day 4 README |
| `kubectl get pods -n dev` | Kubernetes deployment verification |
| `kubectl get svc -n dev` | Kubernetes Service verification |
| `curl http://$APP_DNS/...` | Day 4 README application endpoints |
| `docker manifest inspect abhishekf5/demoservice-b:v` | Docker image verification I performed |
| `kubectl apply -k .` | Kustomize application from the alerting configuration folder |
| `kubectl get servicemonitor,prometheusrule,alertmanagerconfig -n monitoring` | Resource verification used in my practical |
| `kubectl port-forward ...` | Kubernetes port-forwarding method used to access the monitoring UIs |
| `up{namespace="dev"}` | Prometheus query used to verify application scraping |
| `http_requests_total` | Custom metric defined by the application |
| `http_request_duration_seconds` | Custom histogram metric defined by the application |
| `http_request_duration_summary_seconds` | Custom summary metric defined by the application |
| `node_gauge_example` | Custom gauge metric defined by the application |
| `ALERTS` | Prometheus query used to inspect active alert series |
| `kubectl get pods -n dev` | Verification of container restart counts |
| `curl http://$APP_DNS/crash` | Day 4 README alert testing |
| `http://EC2_PUBLIC_IP:9090/alerts` | Prometheus Alerts page used to verify the firing alert |

**My command-source approach:** I followed the Day 4 README, the teacher's practical demonstration and the YAML manifests provided in the repository. I also used additional Kubernetes and Prometheus verification commands during my own execution to confirm that each step worked.

---

## Final Outcome

I successfully implemented custom metrics monitoring and email alerting on Amazon EKS using Prometheus, Grafana and Alertmanager.

I verified that Prometheus collected all four custom application metrics, loaded both alert rules, detected three container restarts and triggered the `PodRestart` alert.

Finally, I received the actual alert notification in Gmail.

**Practical completed successfully.**
