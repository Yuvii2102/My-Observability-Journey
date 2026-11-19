# Day 6 – Distributed Tracing Using Jaeger

## 1. What is Distributed Tracing?

Distributed tracing helps us track a request as it travels through multiple services in a distributed system.

In a microservices architecture, a single user request may pass through multiple services before getting a response.

For example, in an e-commerce application:

1. User sends a login request.
2. Login service authenticates the user.
3. Service A processes the request.
4. Service B communicates with another service.
5. Payment service processes the payment.
6. The final response is returned to the user.

Now imagine the complete request should take 1 second, but it takes 4 seconds.

How do we identify which service is causing the delay?

This is where distributed tracing helps.

It tracks the complete journey of a request and tells us:

- Which services handled the request.
- Which operations were executed.
- How much time each operation took.
- Where the request experienced delays.
- Which service might be causing latency.

**In simple words:** Distributed tracing helps us understand the complete journey of a request across multiple services and identify where the problem is happening.

## 2. Real-World Example of Distributed Tracing

Imagine I am travelling from Hyderabad to Boston.

My journey looks like this:

1. Home to Hyderabad Airport – 45 minutes.
2. Hyderabad to Dubai – 3.5 hours.
3. Dubai to Boston – 12 hours.
4. Boston Airport to my friend's house – 2.5 hours.

My friend usually completes this journey in 23 hours, but my journey took 25 hours.

After checking my travel details, I discovered that the cab journey from Boston Airport to my friend's house took 2 hours longer than expected.

Now I know exactly where the delay happened.

The same concept applies to distributed tracing.

Instead of tracking a travel journey, we track a request travelling between multiple microservices.

Each service records how much time it takes to process the request.

This helps developers and DevOps engineers identify performance issues.

## 3. Why is Distributed Tracing Important?

In modern applications, we use microservices where multiple services communicate with each other.

For example:

- Login Service
- User Service
- Product Service
- Payment Service
- Database Service

If one service becomes slow, the complete application can become slow.

Without distributed tracing, identifying the service responsible for the delay becomes difficult.

Distributed tracing helps us:

1. Identify slow microservices.
2. Troubleshoot application latency.
3. Understand communication between services.
4. Find performance bottlenecks.
5. Debug problems across multiple services.
6. Help developers improve application performance.

## 4. What is Jaeger?

Jaeger is an open-source distributed tracing tool used to monitor and troubleshoot requests in microservices applications.

It helps us collect, store, search and visualize distributed traces.

Using Jaeger, we can:

- Track requests across multiple services.
- Identify slow operations.
- Analyze application latency.
- Visualize service communication.
- Troubleshoot distributed applications.

Jaeger provides a user interface where we can search for traces and understand how requests travel through different services.

## 5. What is OpenTelemetry?

OpenTelemetry is an open-source, vendor-neutral observability framework.

It provides libraries and tools that developers can use to collect and generate telemetry data such as:

- Metrics
- Logs
- Traces

For distributed tracing, developers use OpenTelemetry libraries to instrument their applications.

**What is instrumentation?**

Instrumentation means adding the required code or libraries to an application so that it can generate tracing information.

For example, developers can instrument a Node.js application using OpenTelemetry libraries.

This allows the application to generate trace information whenever a request is processed.

One important advantage of OpenTelemetry is that it is vendor-neutral.

This means we can use the same instrumentation and send tracing data to different supported observability platforms instead of depending on one particular vendor.

## 6. Who is Responsible for Implementing Distributed Tracing?

There are two important responsibilities.

### A. Developers

Developers are responsible for instrumenting the application.

They use OpenTelemetry libraries to generate tracing information.

They can instrument:

- HTTP requests.
- API calls.
- Communication between services.
- Individual functions.
- Database operations.

### B. DevOps / SRE Engineers

DevOps and SRE engineers are responsible for setting up the tracing infrastructure.

Their responsibilities include:

- Setting up the Kubernetes cluster.
- Installing Jaeger.
- Configuring the required components.
- Configuring storage.
- Making the Jaeger UI accessible.
- Maintaining and troubleshooting the tracing infrastructure.

**Important:** Both application instrumentation and tracing infrastructure are required for complete distributed tracing.

Simply installing Jaeger does not automatically provide detailed traces from an application that has not been instrumented.

## 7. Architecture of Jaeger

Jaeger has four main components.

### 1. Agent

The Agent receives tracing information from instrumented applications.

It acts as a collector on the application side and forwards trace data to the Jaeger Collector.

### 2. Collector

The Collector receives trace data from agents.

Its responsibilities include:

- Receiving traces.
- Processing trace information.
- Preparing traces for storage.
- Sending traces to the configured storage backend.

### 3. Storage / Database

Jaeger needs storage to save the collected traces.

Unlike a system with a built-in database, Jaeger supports external storage backends.

Examples include:

- Elasticsearch
- Cassandra

In our practical, we will use Elasticsearch to store the tracing information.

### 4. Query / UI

The Query component retrieves stored traces and provides them to the Jaeger user interface.

Using the UI, we can:

- Search for services.
- Select operations.
- Find traces.
- View individual spans.
- Analyze request execution time.

## 8. Jaeger Architecture Flow

The complete flow looks like this:

Application → Jaeger Agent → Jaeger Collector → Elasticsearch → Jaeger Query / UI

Explanation:

1. The application generates traces using OpenTelemetry instrumentation.
2. The Jaeger Agent receives the trace information.
3. The Agent forwards the traces to the Collector.
4. The Collector processes the traces.
5. The traces are stored in Elasticsearch.
6. Jaeger Query retrieves the stored traces.
7. Users view and analyze the traces through the Jaeger UI.

## 9. What is a Trace?

A trace represents the complete journey of a single request through a distributed application.

For example, when a user makes a payment request, the request might travel through:

Login Service → Service A → Service B → Payment Service.

The complete journey is called a Trace.

A trace helps us understand which services participated in processing a request.

## 10. What is a Span?

A span represents one individual operation or unit of work within a trace.

For example:

- Login Service processing.
- Service A processing.
- Service B processing.
- Payment Service processing.

Each operation can have its own span.

A span contains information such as:

- Operation name.
- Start time.
- End time.
- Duration.
- Service name.
- Related trace and parent span information.

A single trace can contain multiple spans.

**Remember:**

- Trace = Complete journey of a request.
- Span = Individual operation within that journey.

## 11. Difference Between Metrics, Logs and Traces

These are the three important pillars of observability.

| Pillar | What it tells us | Example tool |
|---|---|---|
| Metrics | What is happening? | Prometheus |
| Logs | Why is something happening? | Elasticsearch / Kibana |
| Traces | Where is the request spending time? | Jaeger |

### Metrics

Metrics provide numerical information about the system.

Example: CPU utilization is 90%.

### Logs

Logs provide detailed messages about what happened inside an application.

Example: Database connection timeout.

### Traces

Traces show the complete request journey across different services.

Example: Payment Service is taking 2 seconds to process a request.

All three pillars work together to help us understand and troubleshoot applications.

## 12. Complete Observability Architecture

In our observability setup, we use different tools for different purposes.

| Component | Responsibility | Tool |
|---|---|---|
| Metrics | Collect metrics | Prometheus |
| Alerting | Generate alerts | Alertmanager |
| Metrics Dashboard | Visualize metrics | Grafana |
| Logging | Collect logs | Fluent Bit |
| Log Storage | Store logs | Elasticsearch |
| Log Dashboard | Visualize logs | Kibana |
| Tracing | Track requests | Jaeger |
| Trace Instrumentation | Generate traces | OpenTelemetry |

The architecture works like this:

- Node Exporter collects system metrics and Prometheus scrapes them.
- Fluent Bit collects application logs and forwards them to Elasticsearch.
- OpenTelemetry-instrumented applications generate traces that are sent through the Jaeger tracing pipeline.
- Elasticsearch stores logs and can also store traces.
- Grafana, Kibana and Jaeger provide separate interfaces for analyzing metrics, logs and traces.

## 13. What is Latency?

Latency is the time taken by a request to travel through a system or complete an operation.

For example:

If a payment request takes 2 seconds to complete, its total latency is 2 seconds.

Distributed tracing helps us identify which service or operation is contributing the most latency.

## 14. What is a Performance Bottleneck?

A performance bottleneck is a particular service or operation that slows down the overall application.

For example:

Login Service – 100 ms

Service A – 200 ms

Service B – 2 seconds

Payment Service – 300 ms

Here, Service B is taking significantly more time than the other services.

Using Jaeger, developers can identify this slow operation and investigate how to improve its performance.

## 16. Final Learning

Today I learned about the third pillar of observability, Distributed Tracing.

I understood how requests travel through multiple microservices and how tracing helps identify latency and performance bottlenecks.

I also learned about Jaeger, OpenTelemetry, Agents, Collectors, Storage, Traces and Spans.

The main thing I learned is that developers instrument applications using OpenTelemetry, while DevOps engineers set up and maintain the tracing infrastructure using tools like Jaeger.

**Final Flow:**

Application → OpenTelemetry → Jaeger Agent → Jaeger Collector → Elasticsearch → Jaeger UI
