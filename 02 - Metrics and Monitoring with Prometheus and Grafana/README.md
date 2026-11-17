# Day 2 – Metrics and Monitoring with Prometheus and Grafana

## 1. What are Metrics?

Metrics are periodic measurements collected over time to understand the health and performance of a system.

Let's understand this with a real-life example.

Imagine a patient is admitted to a hospital. A nurse is assigned to take care of the patient.

The nurse checks the patient's health every 15 or 30 minutes, depending on the patient's condition.

The nurse collects important information such as:

- Heartbeat
- Blood pressure
- Body temperature

For example, the nurse records the patient's heartbeat like this:

| Time | Heartbeat |
|---|---:|
| 10:00 AM | 76 |
| 10:15 AM | 81 |
| 10:30 AM | 78 |
| 10:45 AM | 85 |
| 11:00 AM | 72 |

The nurse keeps collecting this information throughout the day.

Why is this important?

Because the doctor needs this historical information to understand whether the patient's health is normal or not.

If the heartbeat suddenly increases or decreases, the doctor can identify the problem and take action.

These periodic measurements are called **Metrics**.

**Definition:** Metrics are historical measurements of events collected over time to understand the health and performance of a system.

The same concept is used in the IT world to understand the health of applications and infrastructure.

---

## 2. Difference Between Metrics and Monitoring

Many people think metrics and monitoring are the same, but they are different.

Let's continue with our hospital example.

### Metrics

Metrics are the raw numerical information collected from a system.

In our hospital example, the heartbeat and blood pressure readings are metrics.

Imagine the nurse has collected hundreds of readings in a notebook or Excel sheet.

It becomes difficult for the doctor to read and understand all the numbers.

Similarly, in IT systems, collecting metrics alone is not enough.

We need a system that can display this information in an easy-to-understand format.

### Monitoring

Monitoring is the process of collecting, visualizing and analyzing metrics to understand the health of a system.

A monitoring system helps us:

1. Collect metrics.
2. Display metrics using graphs and dashboards.
3. Configure alerts when something goes wrong.

For example, imagine the patient's heartbeat goes above 90.

We can configure the monitoring system to send a notification to the nurse.

If the heartbeat goes above 110, it can send a notification to the doctor.

This helps them take action immediately.

Therefore:

- Metrics = Raw numerical information.
- Monitoring = Metrics + Dashboards + Alerts.

**Important:** Metrics are a part of monitoring. Monitoring uses metrics to display information and send alerts.

---

## 3. Understanding Metrics and Monitoring in the IT World

Now let's understand the same concept using our Resume Builder application.

Imagine we have developed a Resume Builder application and deployed it on an AWS EKS Kubernetes cluster.

Our infrastructure contains:

- AWS EC2 instances acting as Kubernetes worker nodes.
- Kubernetes Pods and Deployments.
- AWS networking such as VPC.
- Frontend and backend applications.
- Database.

To understand the health of our complete system, we need to collect different types of metrics.

There are three important categories of metrics.

### A. Infrastructure Metrics

Infrastructure metrics help us understand the health of our infrastructure.

Examples:

- CPU utilization of EC2 instances.
- Memory utilization.
- Disk utilization.
- Network traffic.

For example, we can check the CPU utilization of an EC2 instance at 10 AM, 12 PM and 2 PM.

This helps us understand whether CPU usage suddenly increased at a particular time.

### B. Kubernetes Metrics

Kubernetes metrics help us understand the health of our Kubernetes cluster.

Examples:

- Pod status.
- Deployment status.
- Number of Pod restarts.
- CrashLoopBackOff occurrences.
- Number of replicas in a Deployment.
- Horizontal Pod Autoscaler (HPA) information.

For example, we can check how many times a Pod entered CrashLoopBackOff during the day.

We can also check how many replicas are running in a Deployment.

### C. Application Metrics

Application metrics help us understand the performance and behavior of our applications.

Examples:

- Total HTTP requests received.
- Successful HTTP requests.
- Failed HTTP requests.
- API response time.
- Number of users signing up.
- Number of users deactivating their accounts.
- Time users spend on the application.

For example, in our Resume Builder application, we can collect information about how many users registered during the last 30 days.

We can also understand at what time users usually register or deactivate their accounts.

Depending on the company and application, we can collect hundreds of metrics or just a few important metrics.

There is no fixed number of metrics that every organization must collect.

It depends on what information the organization needs to understand the health of its system.

---

## 4. What is Scraping in Prometheus?

Before understanding Prometheus, we need to understand one important term called **Scraping**.

Scraping means collecting or pulling metrics from different sources.

There are two ways metrics can reach a monitoring system.

### A. Pull Mechanism

In the pull mechanism, the monitoring system periodically contacts the target and collects its metrics.

Prometheus commonly uses this mechanism.

For example, Prometheus contacts an application and collects metrics from its `/metrics` endpoint.

### B. Push Mechanism

In the push mechanism, an application or another component sends its metrics to a receiving system.

Prometheus also supports certain push-based use cases through Pushgateway.

**Remember:** Scraping is the process of pulling metrics from targets. Prometheus mainly follows the pull mechanism.

---

## 5. What is Prometheus?

Prometheus is an open-source monitoring and alerting platform.

It is widely used in Kubernetes environments to collect, store and query metrics.

Prometheus helps us:

- Collect metrics from different sources.
- Monitor applications and infrastructure.
- Store historical metrics.
- Query metrics using PromQL.
- Identify performance problems.
- Configure alerts using Alertmanager.

### Why do we use Prometheus?

Imagine our Resume Builder application is running on a Kubernetes cluster.

We need to collect metrics from:

- AWS EC2 instances.
- Kubernetes nodes.
- Kubernetes Pods.
- Deployments.
- Application endpoints.

Prometheus can collect metrics from these different sources and store them in one place.

We can then query those metrics and understand the health of our complete system.

---

## 6. Prometheus Architecture

Prometheus has different components that work together to collect, store and provide metrics.

The main components are:

1. Prometheus Server.
2. Retrieval.
3. Time Series Database (TSDB).
4. HTTP Server.
5. Pushgateway.
6. Alertmanager.
7. Service Discovery.

### A. Prometheus Server

Prometheus Server is the main component of Prometheus.

It is responsible for collecting and storing metrics.

It contains three important parts:

- Retrieval.
- Time Series Database.
- HTTP Server.

### B. Retrieval

Retrieval is responsible for collecting metrics from different targets.

It periodically scrapes metrics from exporters and other configured targets.

For example:

- Node Exporter provides node-level metrics.
- Kube State Metrics provides Kubernetes resource metrics.
- Application metrics endpoints provide application-level metrics.

Prometheus pulls these metrics and sends them for storage.

### C. Time Series Database (TSDB)

TSDB stands for Time Series Database.

Prometheus stores the collected metrics in its time series database.

A time series database stores data along with timestamps.

For example:

| Timestamp | Metric | Value |
|---|---|---:|
| 10:00 AM | CPU usage | 60% |
| 10:15 AM | CPU usage | 75% |
| 10:30 AM | CPU usage | 90% |
| 10:45 AM | CPU usage | 65% |

Here, we are storing the metric value along with the time at which it was collected.

This helps us understand historical information and identify changes in our system.

### D. HTTP Server

Prometheus provides an HTTP server through which users and other applications can access Prometheus.

Using the Prometheus user interface, we can:

- Execute queries.
- View metrics.
- Check monitoring targets.
- Explore collected information.

Prometheus uses a query language called **PromQL (Prometheus Query Language)**.

We will learn PromQL in Day 3.

### E. Pushgateway

Pushgateway is a component that allows certain applications and short-lived jobs to push their metrics.

Normally, Prometheus pulls metrics from targets.

But imagine we have a batch job that runs for only a few seconds and then exits.

Prometheus might not scrape its metrics while it is running.

In this situation, the application can push its metrics to Pushgateway.

Prometheus then scrapes the metrics from Pushgateway.

**Remember:** Pushgateway is useful for certain short-lived jobs. It is not meant to replace the normal pull mechanism of Prometheus.

### F. Alertmanager

Alertmanager is responsible for handling alerts generated from Prometheus alerting rules.

For example:

Imagine CPU utilization goes above 80%.

We can configure an alerting rule in Prometheus.

When the condition is met, Prometheus sends the alert to Alertmanager.

Alertmanager can then send notifications through:

- Slack.
- Email.
- Other supported notification integrations.

We will learn about Alertmanager and configure alerts in Day 4.

### G. Service Discovery

Service Discovery helps Prometheus identify the targets from which it needs to collect metrics.

Imagine our Kubernetes cluster has 100 applications.

We may want Prometheus to collect metrics from only 50 applications.

Using Service Discovery and target configuration, we can tell Prometheus which applications it should scrape.

This makes metrics collection easier in Kubernetes because Pods and services can be created or deleted frequently.

---

## 7. What are Exporters in Prometheus?

Exporters are components that expose metrics from systems or services in a format that Prometheus can collect.

We can understand exporters as add-ons or plugins that help Prometheus collect information from different sources.

For example, Prometheus cannot directly collect every type of system information by itself.

So we use exporters depending on what we want to monitor.

Our teacher explained three important sources of metrics.

### A. Node Exporter

Node Exporter is used to collect infrastructure-level metrics from machines.

It provides information such as:

- CPU utilization.
- Memory utilization.
- Disk utilization.
- Network statistics.

**How does it work?**

Imagine we have three EC2 instances running as Kubernetes worker nodes.

Node Exporter runs on these nodes and collects system-level information.

It reads information from the machines and exposes the metrics.

Prometheus then scrapes these metrics from Node Exporter.

The flow looks like this:

```text
AWS EC2 / Kubernetes Nodes
          |
          v
     Node Exporter
          |
          v
       Prometheus
          |
          v
    Time Series DB
```

### B. Kube State Metrics

Kube State Metrics is used to collect information about Kubernetes resources.

It communicates with the Kubernetes API Server and exposes information about the state of Kubernetes objects.

Examples:

- Pod status.
- Deployment status.
- Replica information.
- ConfigMaps.
- Secrets metadata.
- Kubernetes resource states.

For example, if a Pod enters CrashLoopBackOff, Kube State Metrics can expose information about its Kubernetes state.

Prometheus scrapes these metrics and stores them.

The flow looks like this:

```text
Kubernetes API Server
          |
          v
    Kube State Metrics
          |
          v
       Prometheus
          |
          v
    Time Series DB
```

**Important difference:**

- Node Exporter collects operating system and machine-level metrics.
- Kube State Metrics exposes Kubernetes object-state metrics.

### C. Application Metrics Endpoint

Now let's understand how we collect metrics from our own applications.

Imagine our Kubernetes cluster contains multiple applications:

- Resume Builder.
- Payment Service.
- User Management Service.
- Product Catalog Service.

We may want to collect application-specific metrics such as:

- Total HTTP requests.
- Failed HTTP requests.
- API response time.
- Number of user registrations.
- Number of user logins.

For this, developers need to instrument their applications using suitable client libraries.

They can expose these metrics through an endpoint, commonly called:

```text
/metrics
```

For example:

```text
http://application-service:8080/metrics
```

Prometheus can scrape this endpoint and collect application metrics.

We can use Service Discovery and target configuration to tell Prometheus which applications it should scrape.

The flow looks like this:

```text
Application
    |
    v
Application Code
    |
    v
Metrics Endpoint (/metrics)
    |
    v
Prometheus
    |
    v
Time Series DB
```

### Summary of the Three Sources

| Component | What does it collect? |
|---|---|
| Node Exporter | EC2 and node-level metrics |
| Kube State Metrics | Kubernetes resource-state metrics |
| Application /metrics endpoint | Application-specific metrics |

These are three important sources of metrics in a Kubernetes monitoring setup.

---

## 8. What is Grafana?

Grafana is an open-source visualization platform used to create dashboards and display metrics.

Prometheus collects and stores metrics, but Grafana makes it easier to understand those metrics using graphs, charts and dashboards.

For example, using Grafana, we can create dashboards showing:

- CPU utilization.
- Memory utilization.
- Disk utilization.
- Kubernetes Pod status.
- Application HTTP requests.
- Application response time.

### How do Prometheus and Grafana work together?

Let's understand the flow:

```text
EC2 / Kubernetes / Applications
              |
              v
      Exporters / Metrics
              |
              v
          Prometheus
              |
              v
      Time Series Database
              |
              v
           Grafana
              |
              v
      Dashboards and Graphs
```

Grafana connects to Prometheus as a data source.

When we create a dashboard in Grafana, it queries Prometheus to retrieve the required metrics.

Grafana then displays those metrics in an easy-to-understand format.

**Remember:**

- Prometheus = Collects and stores metrics.
- Grafana = Visualizes metrics using dashboards.

---

## 9. Other Monitoring Tools

Prometheus is not the only monitoring tool available.

There are other monitoring tools used by organizations.

| Tool | Description |
|---|---|
| Prometheus | Open-source monitoring and alerting |
| Nagios | Infrastructure and service monitoring |
| InfluxDB | Time series database |
| Graphite | Metrics storage and visualization ecosystem |

Prometheus is widely used in Kubernetes environments because it integrates well with Kubernetes and has a strong open-source community.

It is also a graduated project of the Cloud Native Computing Foundation (CNCF).

---

## 10. How We Will Install Prometheus in Kubernetes

Our teacher uses an AWS EKS cluster to demonstrate Prometheus and Grafana.

We will follow the same approach during the practical session.

The main components we will install are:

1. Prometheus.
2. Grafana.
3. Alertmanager.
4. Node Exporter.
5. Kube State Metrics.

### Why do we use Helm?

Helm is a package manager for Kubernetes.

Instead of manually creating multiple Kubernetes YAML files for every component, we can use Helm charts to install and manage applications on Kubernetes.

Our teacher uses a Prometheus Helm chart to install the monitoring stack.

During the practical session, we will learn how to:

- Add the Prometheus Helm repository.
- Create a monitoring namespace.
- Install Prometheus and Grafana.
- Access Prometheus UI.
- Access Grafana UI.
- Access Alertmanager UI.
- Configure Prometheus as a Grafana data source.

### How will we access the UIs?

During the demonstration, our teacher uses Kubernetes port forwarding.

For example:

```bash
kubectl port-forward
```

This allows us to access the services locally without configuring an Ingress Controller.

Later, we can learn how to expose these services using Ingress.

---

## 11. Quick Revision

| Topic | What I Learned |
|---|---|
| Metrics | Periodic numerical measurements |
| Monitoring | Collecting, visualizing and alerting on metrics |
| Scraping | Pulling metrics from targets |
| Prometheus | Collects and stores metrics |
| Retrieval | Collects metrics from targets |
| TSDB | Stores time-series data |
| HTTP Server | Provides access to Prometheus |
| Pushgateway | Receives pushed metrics from certain jobs |
| Alertmanager | Handles and routes alerts |
| Service Discovery | Identifies monitoring targets |
| Node Exporter | Collects machine-level metrics |
| Kube State Metrics | Exposes Kubernetes resource-state metrics |
| Metrics Endpoint | Exposes application-specific metrics |
| Grafana | Creates dashboards and visualizes metrics |
| Helm | Installs and manages Kubernetes applications |

## 12. My Understanding

Today I learned about the first pillar of observability, which is Metrics, and understood the difference between metrics and monitoring.

I started by understanding how a nurse collects a patient's heartbeat and blood pressure periodically. I understood that these measurements are metrics and that a monitoring system helps doctors visualize this information and receive alerts when something goes wrong.

Then I connected the same concept to the IT world and learned how we collect metrics from AWS infrastructure, Kubernetes clusters and applications.

I also learned about Prometheus architecture and its important components, including Retrieval, Time Series Database, HTTP Server, Pushgateway and Alertmanager.

I understood the role of Node Exporter, Kube State Metrics and application metrics endpoints in collecting different types of metrics.

Finally, I learned how Grafana connects with Prometheus to display metrics using dashboards and graphs.

In the upcoming practical session, I will install Prometheus and Grafana on a Kubernetes cluster and understand how they work together.
