# Day 4 – Custom Metrics Instrumentation and Alertmanager

## Objective

In this lesson, I learned about custom metrics instrumentation, Prometheus metric types, ServiceMonitor and Alertmanager.

I also understood how applications expose their own metrics and how Prometheus discovers those metrics.

## 1. What is Instrumentation?

Instrumentation is the process of adding code to an application to collect and expose useful information about its behaviour.

For example, in an e-commerce application, I may want to monitor:

- Total number of user logins
- Number of accounts created
- Total HTTP requests received
- HTTP request duration
- Application-specific errors

Prometheus cannot automatically understand all these application-specific metrics.

Developers need to instrument their applications so that these metrics can be collected.

This process is called instrumentation.

## 2. Why is Instrumentation Important?

Even if an organisation has configured Prometheus, Grafana, EFK Stack and Jaeger, the observability setup cannot provide application-specific information if the applications do not expose metrics, logs or traces.

Observability is a collective responsibility.

- Developers instrument applications.
- DevOps and SRE engineers configure monitoring infrastructure.
- Prometheus collects metrics.
- Grafana visualises metrics.
- Alertmanager handles alerts.

Instrumentation is important because it provides the information needed to understand application behaviour.

## 3. Why Can't Exporters Collect Everything?

Exporters collect specific types of metrics.

Examples:

- Node Exporter collects CPU, memory and disk-related metrics from nodes.
- MySQL Exporter collects database-related metrics.

However, exporters cannot automatically collect every application-specific metric.

For example, Node Exporter cannot tell me how many users registered in my application.

For such information, developers need to instrument the application.

## 4. Four Types of Prometheus Metrics

Prometheus supports four important metric types:

### A. Counter

A Counter is a metric that only increases over time.

Examples:

- Total HTTP requests
- Total user logins
- Total accounts created

A Counter increases whenever a particular event occurs.

### B. Gauge

A Gauge is a metric that can increase or decrease.

Examples:

- CPU utilisation
- Memory utilisation
- Number of active users
- Active connections

Gauge is useful for values that continuously change.

### C. Histogram

A Histogram measures observations and groups them into predefined buckets.

It is commonly used for:

- HTTP request duration
- API response time
- Request latency

For example, I can create buckets for request durations of 0.5 seconds, 1 second, 5 seconds and 10 seconds.

This helps me understand the distribution of request durations.

### D. Summary

A Summary is another metric type used to measure observations such as request duration and latency.

It provides statistical information about observed values, including quantiles.

My teacher introduced Summary but did not cover its details deeply in this lesson.

### Quick Comparison

| Metric Type | Purpose | Example |
|---|---|---|
| Counter | Tracks increasing values | Total HTTP requests |
| Gauge | Tracks values that increase or decrease | CPU utilisation |
| Histogram | Groups observations into buckets | HTTP request duration |
| Summary | Provides statistical information | Request latency |

## 5. Instrumentation Using Node.js

My teacher demonstrated custom metrics instrumentation using a Node.js application.

The application uses a Prometheus client to define custom metrics.

Examples:

- `http_requests_total` – Counter
- `http_request_duration_seconds` – Histogram

The application updates these metrics whenever relevant events occur.

It exposes the metrics through an endpoint:

`/metrics`

Prometheus can collect metrics from this endpoint once the application is discovered and configured as a scraping target.

## 6. What is Service Discovery?

Service Discovery helps Prometheus identify the applications and endpoints from which it should collect metrics.

In Kubernetes, many applications can run inside the same cluster.

Each application may expose its own `/metrics` endpoint.

Prometheus needs to know which applications it should monitor.

This is where Service Discovery is used.

## 7. What is ServiceMonitor?

ServiceMonitor is a Kubernetes custom resource provided by the Prometheus Operator.

It tells Prometheus which Kubernetes Services it should monitor and which metrics endpoint it should scrape.

For example, an application may have five services:

- Login Service
- Payment Service
- Product Service
- Cart Service
- Logout Service

If only Login Service and Payment Service expose application metrics, ServiceMonitor can select those Services for monitoring.

### ServiceMonitor Workflow

1. Developers instrument the application.
2. The application exposes metrics through `/metrics`.
3. A Kubernetes Service provides access to the application.
4. ServiceMonitor selects the required Service.
5. Prometheus discovers the Service and scrapes its metrics.
6. Grafana visualises the collected metrics.

Instrumentation alone is not enough. Prometheus also needs to discover the application before it can collect its custom metrics.

## 8. What is Alertmanager?

Alertmanager is a component used with Prometheus to handle alerts.

Prometheus evaluates alerting rules and sends alerts to Alertmanager when their conditions are met.

Alertmanager processes and routes these alerts to configured receivers.

Examples of situations that may generate alerts:

- High CPU usage
- Container crashes
- Repeated Pod restarts
- Application failures

## 9. How Alertmanager Works

1. An application or Kubernetes resource experiences a problem.
2. Prometheus collects the relevant metrics.
3. Prometheus evaluates its alerting rules.
4. Prometheus sends the alert to Alertmanager.
5. Alertmanager processes and routes the alert.
6. A notification is sent to the configured receiver.

## 10. Configure Email Notifications

My teacher demonstrated sending email notifications through Gmail.

Alertmanager needs SMTP configuration to send emails.

The configuration includes:

- Receiver email address
- Sender email address
- SMTP server
- SMTP username
- SMTP password

For Gmail, an App Password is used instead of the normal account password.

The configuration process involves:

1. Enabling two-factor authentication.
2. Generating a Gmail App Password.
3. Encoding the password using Base64 for the Kubernetes Secret configuration.
4. Updating the email Secrets YAML file.
5. Updating the Alertmanager configuration with the receiver email address.
6. Applying the updated Kubernetes manifests.

Secrets must be protected and should never be committed to a public GitHub repository.

## 11. Trigger Alerts by Crashing an Application

My teacher demonstrated alerting using a sample Node.js application.

The application provides a `/crash` endpoint.

When this endpoint is accessed, the application crashes.

Kubernetes detects the failed container and restarts it according to its restart policy.

Prometheus collects the restart metrics and evaluates the configured alerting rules.

When the alert condition is met, Prometheus sends the alert to Alertmanager.

Alertmanager then sends an email notification to the configured receiver.

This demonstrates how monitoring and alerting work together.

## 12. What I Learned

In this lesson, I learned:

- What instrumentation means and why it is important.
- Why exporters cannot collect every application-specific metric.
- The four Prometheus metric types: Counter, Gauge, Histogram and Summary.
- How developers expose custom metrics using a Node.js application.
- Why Prometheus needs Service Discovery.
- How ServiceMonitor helps Prometheus discover application metrics.
- What Alertmanager does.
- How email notifications can be configured.
- How application crashes can trigger alerts.

## References

- Teacher: Abhishek Veeramalla
- GitHub Repository: https://github.com/iam-veeramalla/observability-zero-to-hero
- Prometheus Documentation: https://prometheus.io/docs/
- Prometheus Metric Types: https://prometheus.io/docs/concepts/metric_types/
- Prometheus Operator: https://github.com/prometheus-operator/prometheus-operator
- Alertmanager: https://prometheus.io/docs/alerting/latest/alertmanager/
