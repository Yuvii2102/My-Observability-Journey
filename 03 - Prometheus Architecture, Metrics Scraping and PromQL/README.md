# Day 3 – Prometheus Architecture, Metrics Scraping and PromQL

## Introduction

In Day 3 of the Observability Zero to Hero series, I learned how Prometheus collects metrics from different sources in a Kubernetes cluster and how we can query those metrics using PromQL.

In Day 2, I installed Prometheus, Grafana and Alertmanager on an AWS EKS cluster using Helm.

In this session, I focused more on understanding how Prometheus works internally, how it collects metrics from Node Exporter and Kube State Metrics, how to write basic PromQL queries and how Grafana helps us visualize metrics using dashboards.

## 1. Prometheus Architecture

Prometheus is a monitoring system that collects metrics from different sources and stores them in a Time Series Database.

The main responsibility of Prometheus is to scrape metrics from different exporters and applications.

The primary sources from which Prometheus collects metrics are:

1. Node Exporter
2. Kube State Metrics
3. Custom Application Metrics

After collecting the metrics, Prometheus stores them in its Time Series Database. We can then use PromQL to query and analyze the information.

### Basic Prometheus Architecture

```text
             Node Exporter
                   |
                   |
             Kube State Metrics
                   |
                   |
             Custom Metrics
                   |
                   |
                   v
              Prometheus
                   |
                   v
          Time Series Database
                   |
                   v
                PromQL
                   |
                   v
           Prometheus UI
                   |
                   v
                Grafana
```

Prometheus also integrates with Alertmanager to handle alerts when configured conditions are met.

## 2. Node Exporter

Node Exporter is an exporter that collects infrastructure-level metrics from Kubernetes worker nodes.

In AWS, our Kubernetes worker nodes are EC2 instances. Node Exporter collects information about these machines.

Some examples of metrics collected by Node Exporter are:

- CPU usage
- Memory usage
- Disk usage
- Network information
- Running processes

### How does Node Exporter work?

Node Exporter runs as a Pod on every eligible Kubernetes node.

It collects information from the system files and exposes the metrics through an endpoint called `/metrics`.

For example:

```text
http://<NODE-EXPORTER-IP>:9100/metrics
```

When we access this endpoint, we can see the metrics collected by Node Exporter.

Prometheus periodically scrapes this endpoint and stores the collected information.

### Why does Node Exporter run as a DaemonSet?

Node Exporter needs to collect information from every Kubernetes worker node.

A DaemonSet ensures that a Node Exporter Pod runs on each eligible node.

For example, if our EKS cluster has two worker nodes, Node Exporter runs on both nodes.

## 3. Kube State Metrics

Kube State Metrics is another important source from which Prometheus collects metrics.

It communicates with the Kubernetes API Server and collects information about the current state of Kubernetes resources.

Some examples of information collected by Kube State Metrics are:

- Pod status
- Deployment status
- ReplicaSet information
- Service information
- ConfigMaps
- Secrets
- Container restart counts

### How does Kube State Metrics work?

Kube State Metrics continuously communicates with the Kubernetes API Server and collects information about Kubernetes resources.

It exposes this information through its `/metrics` endpoint.

Prometheus periodically scrapes that endpoint and stores the metrics in its Time Series Database.

### Why does Kube State Metrics need only one replica?

Unlike Node Exporter, Kube State Metrics does not need to run on every worker node.

It communicates with the Kubernetes API Server, which provides information about resources across the cluster.

Therefore, one replica can collect information about the entire Kubernetes cluster.

## 4. Custom Application Metrics

Apart from infrastructure and Kubernetes metrics, Prometheus can also collect custom metrics from applications.

Custom metrics are metrics created to monitor application-specific activities.

For example, consider an e-commerce application.

We might want to monitor:

- Number of users registered
- Number of login requests
- Number of successful payments
- Number of failed transactions
- Time taken to process HTTP requests

Developers add instrumentation to their applications to expose these metrics.

Prometheus can then scrape the application metrics through its `/metrics` endpoint.

We will learn how to create custom metrics using OpenTelemetry in the upcoming sessions.

## 5. What is the /metrics Endpoint?

Exporters and instrumented applications expose their collected metrics through an endpoint called `/metrics`.

For example:

```text
http://<NODE-EXPORTER-IP>:9100/metrics
```

When we send a request to this endpoint, the exporter returns the metrics it has collected.

The information is exposed in a format that Prometheus understands.

Prometheus periodically visits these endpoints and collects the available metrics.

## 6. What is Scraping?

Scraping is the process in which Prometheus periodically pulls metrics from exporters and configured application endpoints.

Let's understand it with a simple example:

1. Node Exporter collects CPU and memory information from a worker node.
2. It exposes the information through the `/metrics` endpoint.
3. Prometheus visits the endpoint.
4. Prometheus collects the metrics.
5. The metrics are stored in the Time Series Database.

The same process happens with Kube State Metrics and other configured metric sources.

This is how Prometheus collects information from different parts of our Kubernetes cluster.

## 7. What is a Time Series Database (TSDB)?

Prometheus stores the collected metrics in a Time Series Database.

A time series is a collection of metric values recorded along with their timestamps.

For example, consider CPU usage on a worker node:

| Time | CPU Usage |
|---|---:|
| 10:00 AM | 25% |
| 10:01 AM | 35% |
| 10:02 AM | 45% |
| 10:03 AM | 30% |

Prometheus stores the metric values along with their timestamps.

This helps us understand how a metric changes over time.

Instead of looking at only the current CPU usage, we can also see its historical values and understand whether CPU usage is increasing or decreasing.

## 8. What is PromQL?

PromQL stands for Prometheus Query Language.

It is the query language used to retrieve and analyze metrics stored in Prometheus.

We can compare it with SQL:

- SQL is used to query relational databases.
- PromQL is used to query Prometheus metrics.

Prometheus collects a large amount of information. Reading all the raw metrics is difficult.

PromQL helps us select specific metrics, filter information and perform calculations.

### Example 1: Get container restart information

```promql
kube_pod_container_status_restarts_total
```

This query returns the total restart count of containers across the available metric series.

### Example 2: Filter metrics by namespace

```promql
kube_pod_container_status_restarts_total{namespace="default"}
```

This query returns container restart information only from the `default` namespace.

### Example 3: Filter metrics by Pod

```promql
kube_pod_container_status_restarts_total{pod="busybox-crash"}
```

This query returns restart information for the specified Pod.

## 9. Understanding Labels in PromQL

Labels help us filter and identify specific metric series.

For example:

```promql
kube_pod_container_status_restarts_total{namespace="default", pod="busybox-crash"}
```

Here:

- `namespace` is a label.
- `pod` is another label.
- `default` is the namespace value.
- `busybox-crash` is the Pod name.

Using labels, we can retrieve information about a particular Pod, namespace or other Kubernetes resource.

We can also remove the namespace filter to retrieve information across all namespaces.

## 10. Table and Graph Views in Prometheus

After executing a PromQL query, Prometheus can display the results in different formats.

### Table View

Table view displays the metric names, labels and their values.

It is useful when we want to see the actual metric information.

### Graph View

Graph view displays how metric values change over time.

It is useful when we want to understand the behaviour of a metric over a specific period.

We can also change the time range to view metrics over the last few minutes, hours or days.

For example, if we want to monitor container restarts, we can use the Table view to see the restart count and the Graph view to understand when the restarts happened.

## 11. Practical Example – Monitoring a Crashing Pod

My teacher demonstrated how Prometheus can monitor a Pod that continuously crashes.

First, we create a Pod that keeps exiting with an error.

```bash
kubectl run busybox-crash \
  --image=busybox \
  --restart=Never \
  -- /bin/sh -c "exit 1"
```

The Pod repeatedly fails and Kubernetes reports its status as `CrashLoopBackOff` after the applicable restart attempts.

Kube State Metrics collects information about the container restarts.

Prometheus scrapes this information and stores it.

We can then execute the following PromQL query:

```promql
kube_pod_container_status_restarts_total{namespace="default"}
```

This query returns the container restart information from the default namespace.

We can open the Graph view to see how the restart count changes over time.

We can also change the time range to understand how many times the Pod restarted during a particular period.

### How does the information reach Prometheus?

The process happens like this:

1. I create a Pod using kubectl.
2. The request goes to the Kubernetes API Server.
3. Kubernetes creates the Pod.
4. Kube State Metrics observes the Kubernetes resource information.
5. It exposes the metrics through its `/metrics` endpoint.
6. Prometheus scrapes the endpoint.
7. Prometheus stores the metrics in its Time Series Database.
8. I execute a PromQL query to retrieve the information.

This helps us monitor Pod restarts without manually checking the Kubernetes cluster every time.

## 12. Aggregation Functions in PromQL

Sometimes we don't want to see individual metric values. Instead, we want to calculate a combined result.

PromQL provides aggregation functions to perform these calculations.

Two important functions are `sum()` and `avg()`.

### A. sum()

The `sum()` function adds multiple metric values.

Example:

```promql
sum(kube_pod_container_status_restarts_total)
```

This calculates the total container restart count across the selected metric series.

We can also group the results by namespace:

```promql
sum by (namespace) (kube_pod_container_status_restarts_total)
```

This calculates the total container restart count separately for each namespace.

### B. avg()

The `avg()` function calculates the average of metric values.

Example:

```promql
avg(kube_pod_container_status_restarts_total)
```

This calculates the average restart count across the selected metric series.

These functions are useful when we want to understand the overall status of our Kubernetes cluster instead of checking each metric individually.

## 13. Why Do We Need Grafana?

Prometheus already provides a basic user interface where we can execute queries and view graphs.

So why do we need Grafana?

Grafana is a visualization and dashboard platform that provides better ways to display and understand monitoring data.

It connects to Prometheus and uses PromQL queries to retrieve metrics.

### Benefits of Grafana

**1. Better Visualization**

Grafana provides dashboards, graphs and charts that make monitoring information easier to understand.

**2. Prebuilt Dashboards**

Grafana provides dashboards for commonly used metrics such as:

- Node CPU usage
- Node memory usage
- Pod resource utilization
- Namespace resource utilization
- Persistent volume information

**3. Multiple Data Sources**

Grafana supports multiple data sources such as:

- Prometheus
- InfluxDB
- Graphite
- Other supported monitoring systems

**4. Authentication and Authorization**

Grafana supports user management and access permissions.

For example:

- Management can have permission to view dashboards.
- DevOps engineers can create and update dashboards.
- Developers can access dashboards related to their applications.
- QA teams can view the information they need.

Grafana can also integrate with authentication systems such as SSO.

**5. Dashboard Sharing**

We can create dashboards and share them with our teams and management.

This allows everyone to view important metrics without manually executing PromQL queries.

## 14. Grafana Data Sources

A data source tells Grafana where to get its monitoring information.

In our practical, we installed Prometheus and Grafana in the same Kubernetes cluster.

Grafana uses Prometheus as its data source.

The process is:

1. Prometheus collects and stores metrics.
2. Grafana connects to Prometheus.
3. Grafana uses PromQL queries to retrieve the metrics.
4. Grafana displays the information through dashboards.

Grafana is not limited to Prometheus. We can also connect other supported data sources such as InfluxDB and Graphite.

This makes Grafana useful even if an organization changes its monitoring system.

## 15. Grafana Dashboards

Grafana provides prebuilt dashboards that help us monitor our Kubernetes cluster.

Some examples include:

- Node resource utilization
- Pod CPU usage
- Pod memory usage
- Namespace resource utilization
- Persistent volume information

We can also create our own dashboards using PromQL queries.

For example, suppose we want to monitor Pod restarts in the default namespace.

We can use:

```promql
kube_pod_container_status_restarts_total{namespace="default"}
```

We select Prometheus as the data source, enter the query and create a dashboard panel.

After creating the panel, we can save the dashboard.

Our management or team members can then access the dashboard and monitor the information whenever they need it.

## 16. What is ServiceMonitor?

ServiceMonitor is a Kubernetes custom resource provided by the Prometheus Operator.

It tells Prometheus which Kubernetes Services should be monitored and scraped for metrics.

Imagine we have an e-commerce application running on Kubernetes with several Services:

- Login Service
- Payment Service
- Product Service
- Order Service

Suppose only the Login Service and Payment Service expose custom metrics through their `/metrics` endpoints.

Should Prometheus scrape every Service in the cluster?

No. We only need to monitor the Services that expose the metrics we want.

We can create a ServiceMonitor to select the required Services.

For example, ServiceMonitor can tell Prometheus to scrape the Login Service and Payment Service.

This avoids unnecessary scraping of Services that are not required for monitoring.

We will learn more about ServiceMonitor and custom metrics in the upcoming practical sessions.

---

# Quick Revision

| Concept | My Understanding |
|---|---|
| Node Exporter | Collects infrastructure metrics from worker nodes |
| Kube State Metrics | Collects Kubernetes resource information |
| Custom Metrics | Application-specific metrics |
| `/metrics` | Endpoint that exposes metrics |
| Scraping | Prometheus periodically pulls metrics |
| TSDB | Stores metrics along with timestamps |
| PromQL | Query language used by Prometheus |
| Labels | Help filter metric information |
| `sum()` | Adds metric values |
| `avg()` | Calculates average metric values |
| Grafana | Visualization and dashboard platform |
| Data Source | Connects Grafana to a monitoring system |
| ServiceMonitor | Selects Kubernetes Services for metric scraping |

# What I Learned

In this session, I learned how Prometheus collects metrics from different sources such as Node Exporter, Kube State Metrics and custom application metrics.

I understood how Node Exporter collects infrastructure information from worker nodes and how Kube State Metrics gets information about Kubernetes resources through the API Server.

I also learned about the `/metrics` endpoint, scraping and how Prometheus stores metric values with timestamps in its Time Series Database.

I learned the basics of PromQL, including how to query metrics, filter them using labels and perform calculations using aggregation functions such as `sum()` and `avg()`.

I also understood the difference between Prometheus and Grafana. Prometheus collects and queries metrics, while Grafana provides better visualization through dashboards.

Finally, I learned about ServiceMonitor and how it helps Prometheus select the Kubernetes Services that need to be monitored.

This session helped me understand how Prometheus works internally and how I can use metrics to monitor the health of a Kubernetes cluster.

---

# References and Sources

- Teacher: Abhishek Veeramalla
- Course: Observability Zero to Hero – Day 3
- GitHub Repository: https://github.com/iam-veeramalla/observability-zero-to-hero
- Prometheus Documentation: https://prometheus.io/docs/
- PromQL Documentation: https://prometheus.io/docs/prometheus/latest/querying/basics/
- Grafana Documentation: https://grafana.com/docs/
- Kube State Metrics: https://github.com/kubernetes/kube-state-metrics
- Node Exporter: https://github.com/prometheus/node_exporter
