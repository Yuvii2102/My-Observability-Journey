# Day 5 – Logging and EFK Stack

## 1. Introduction to Logging

In the previous four episodes of the Observability Zero to Hero series, I learned about the first pillar of observability: **Metrics**.

Now, I am moving to the second pillar of observability: **Logs**.

Logging is an important part of monitoring and troubleshooting applications, especially in distributed systems where multiple applications and microservices run across different servers and Kubernetes nodes.

### What are Logs?

Logs are messages written by developers inside an application or program to explain what the application is doing at a particular point in time.

These messages help us understand:

- What is happening inside an application?
- Which operation is currently running?
- Did an error occur?
- Why did the application fail?
- Where exactly did the problem happen?

For example, consider a simple application that adds two numbers.

Without logs, a user might not understand what input the application expects or what operation it is performing.

With logs, the application can display messages such as:

```text
Starting the addition process...
Reading the first number...
Reading the second number...
Addition process completed.
Output: 30
```

These messages help users and developers understand what the application is doing.

### Why are Logs Important?

Imagine an application contains 10,000 lines of code. Suddenly, the application stops working.

Without logs, it becomes difficult to identify:

- Where the application failed.
- Which operation caused the failure.
- Whether an exception occurred.
- What caused the application to stop working.

Logs help developers identify the exact point where something went wrong and troubleshoot the issue.

This is why developers add log messages at important stages of an application.

---

## 2. Three Pillars of Observability

Observability has three important pillars:

1. Metrics
2. Logs
3. Traces

Each pillar helps us understand a different aspect of an application or system.

| Pillar | What it tells us |
|---|---|
| Metrics | What is happening with the system? |
| Logs | Why is something happening or failing? |
| Traces | How does a request travel through different services? |

### 1. Metrics

Metrics provide numerical information about the system's performance and health.

Examples:

- CPU usage
- Memory usage
- Request count
- Application response time

Tools used in our previous practicals include Prometheus and Grafana.

### 2. Logs

Logs provide detailed messages about events happening inside an application.

Examples:

- Application started successfully.
- Database connection failed.
- User login successful.
- Internal server error occurred.

Logs help us investigate why an application is behaving unexpectedly.

### 3. Traces

Traces help us understand how a request travels through different services in a distributed application.

For example, when a user makes a request:

```text
User Request
     |
     v
Frontend Service
     |
     v
Backend Service
     |
     v
Database Service
     |
     v
Response
```

Tracing helps us understand the complete request flow and identify where delays or failures occur.

**Simple way to remember:**

- Metrics tell us what happened.
- Logs help us understand why it happened.
- Traces help us understand how it happened across services.

---

## 3. What is Centralized Logging?

Before understanding EFK, I need to understand why centralized logging is required.

Imagine an organization has 100 microservices running inside a Kubernetes cluster.

These applications may run in:

- Different namespaces.
- Different pods.
- Different worker nodes.
- Different environments.

Now imagine some applications are experiencing database connection problems.

For example:

```text
Service 1  → Working
Service 2  → Database connection failed
Service 3  → Working
Service 4  → Working
Service 5  → Database connection failed
Service 6  → Working
```

If logs are stored separately in every application, I need to access each pod and check its logs individually.

For 100 applications, this becomes difficult and time-consuming.

### What is the Solution?

Instead of checking logs from every application separately, I can create a **Centralized Logging System**.

A centralized logging system collects logs from multiple applications and stores them in one central location.

I can then search all the collected logs from a single interface.

### Advantages of Centralized Logging

1. All application logs are available in one place.
2. Troubleshooting becomes easier.
3. I can search for errors across multiple applications.
4. I can identify database connection problems quickly.
5. I can investigate issues across different Kubernetes namespaces.
6. It helps organizations respond faster to critical application failures and vulnerabilities.

For example, instead of checking 100 different pods, I can search for:

```text
database connection timeout
```

The centralized logging system can help me identify which services generated that error.

---

## 4. What is the EFK Stack?

EFK is a popular centralized logging stack used to collect, store and visualize application logs.

EFK stands for:

- **E – Elasticsearch**
- **F – Fluent Bit**
- **K – Kibana**

Each component performs a different job.

| Component | Responsibility |
|---|---|
| Elasticsearch | Stores and indexes logs |
| Fluent Bit | Collects and forwards logs |
| Kibana | Provides a user interface to search and visualize logs |

Let's understand each component.

---

## 5. Elasticsearch

Elasticsearch is the storage and search component of the EFK stack.

It stores and indexes the logs collected from different applications.

### What Does Elasticsearch Do?

1. Receives logs from Fluent Bit.
2. Stores the logs.
3. Indexes the logs.
4. Allows users to search and retrieve logs.

Imagine Fluent Bit collects logs from 100 applications.

Instead of storing these logs separately, it forwards them to Elasticsearch.

Elasticsearch stores these logs so that users can search them later.

### Why is Persistent Storage Important?

Application logs can consume a large amount of storage.

For example, an organization may generate several gigabytes of logs every day.

If Elasticsearch runs inside Kubernetes, its data must be stored persistently so that it is not lost when a pod restarts or gets replaced.

In this practical, Elasticsearch uses Kubernetes StatefulSet-based deployment with persistent storage backed by AWS EBS volumes.

The teacher also explains the use of EBS snapshots for backing up stored data.

### Important Points

- Elasticsearch stores and indexes logs.
- It supports searching and retrieving logs.
- It acts as the central storage component of EFK.
- Persistent storage helps preserve logs across pod restarts.

---

## 6. Fluent Bit

Fluent Bit is the log collection and forwarding component of the EFK stack.

It collects logs from Kubernetes nodes and forwards them to Elasticsearch.

### What Does Fluent Bit Do?

In Kubernetes, applications run inside pods.

These pods generate logs while executing.

Fluent Bit collects these container logs and sends them to Elasticsearch.

### Why is Fluent Bit Deployed as a DaemonSet?

A DaemonSet ensures that a Fluent Bit pod runs on every eligible Kubernetes node.

For example, imagine I have two worker nodes:

```text
          Kubernetes Cluster
                  |
          -------------------
          |                 |
          v                 v
       Node 1             Node 2
          |                 |
          v                 v
     Fluent Bit        Fluent Bit
          |                 |
          v                 v
     Collect Logs       Collect Logs
          |                 |
          -------------------
                  |
                  v
             Elasticsearch
```

Each Fluent Bit instance collects logs from the node where it is running.

This is why Fluent Bit is deployed as a DaemonSet rather than as a normal Deployment.

### What is a DaemonSet?

A DaemonSet is a Kubernetes resource that ensures a pod runs on every eligible node in a cluster.

For example:

| Kubernetes Nodes | Fluent Bit Pods |
|---|---|
| Node 1 | Fluent Bit Pod 1 |
| Node 2 | Fluent Bit Pod 2 |
| Node 3 | Fluent Bit Pod 3 |

If there are two eligible worker nodes, I generally expect two Fluent Bit pods.

### Advantages of Fluent Bit

- Lightweight.
- Collects logs from different sources.
- Forwards logs to Elasticsearch.
- Runs as a DaemonSet in Kubernetes.
- Supports different output destinations.
- Can filter logs before forwarding them.

---

## 7. Kibana

Kibana is the visualization and user interface component of the EFK stack.

It connects to Elasticsearch and allows users to search, explore and visualize stored logs.

### What Does Kibana Do?

Imagine Fluent Bit collects application logs and Elasticsearch stores them.

But how can developers easily search through thousands or millions of log entries?

This is where Kibana comes in.

Kibana provides a graphical interface where users can:

- View application logs.
- Search for specific error messages.
- Filter logs by namespace.
- Filter logs by application.
- Examine timestamps.
- Investigate application failures.

### Example

Suppose I want to find database connection errors.

I can open Kibana and search for:

```text
database connection timeout
```

Kibana retrieves matching log information from Elasticsearch and displays the results.

### Important Points

- Kibana is the visualization component.
- It communicates with Elasticsearch.
- It provides a graphical interface for searching logs.
- It supports filtering and exploring log data.

---

## 8. EFK Architecture

Now that I understand the three components, I can understand how they communicate with each other.

### Architecture Diagram

```text
           Kubernetes Cluster
                    |
        -------------------------
        |           |           |
        v           v           v
     Node 1      Node 2      Node 3
        |           |           |
        v           v           v
    Fluent Bit  Fluent Bit  Fluent Bit
        |           |           |
        ----------- | -----------
                    |
                    v
              Elasticsearch
               (Storage)
                    |
                    v
                 Kibana
             (Visualization)
                    |
                    v
                  User
```

### Step-by-Step Working

**Step 1: Applications Generate Logs**

Applications running inside Kubernetes pods generate logs.

**Step 2: Fluent Bit Collects Logs**

Fluent Bit runs as a DaemonSet on Kubernetes nodes and collects container logs.

**Step 3: Fluent Bit Forwards Logs**

Fluent Bit forwards the collected logs to Elasticsearch.

**Step 4: Elasticsearch Stores Logs**

Elasticsearch stores and indexes the received logs.

**Step 5: Kibana Retrieves Logs**

Kibana communicates with Elasticsearch and retrieves the required log information.

**Step 6: Users Search Logs**

Developers and DevOps engineers use the Kibana interface to search and investigate application logs.

### Complete Flow

```text
Applications
     |
     v
Container Logs
     |
     v
Fluent Bit
     |
     v
Elasticsearch
     |
     v
Kibana
     |
     v
Search and Troubleshooting
```

---

## 9. Difference Between EFK and ELK Stack

Another popular logging stack is ELK.

ELK stands for:

- Elasticsearch
- Logstash
- Kibana

The main difference between EFK and ELK is the log collection and processing component.

| Feature | EFK Stack | ELK Stack |
|---|---|---|
| E | Elasticsearch | Elasticsearch |
| F / L | Fluent Bit | Logstash |
| K | Kibana | Kibana |
| Main purpose | Collects and forwards logs | Collects, processes and forwards logs |
| Resource usage | Fluent Bit is lightweight | Logstash generally has a heavier footprint |
| Processing | Supports filtering and processing | Provides extensive processing capabilities |

### What is Logstash?

Logstash is a data processing pipeline used to collect, transform and forward data.

It can perform advanced operations such as:

- Filtering logs.
- Modifying log fields.
- Parsing log messages.
- Transforming data.
- Forwarding processed data to different destinations.

### Why Did the Teacher Choose EFK Instead of ELK?

The teacher explained that many organizations do not need the advanced processing features provided by Logstash.

If the main requirement is to collect logs from Kubernetes nodes and forward them to Elasticsearch, Fluent Bit is sufficient.

Fluent Bit is lightweight and uses fewer resources compared with Logstash in many common deployments.

Therefore, the teacher chose EFK for this practical.

**Simple difference to remember:**

- Fluent Bit is mainly used as a lightweight log collector and forwarder.
- Logstash provides a more feature-rich log processing pipeline.

---

## 10. What is AWS EBS?

AWS EBS stands for Elastic Block Store.

It provides block storage volumes that can be attached to Amazon EC2 instances.

In this practical, EBS is used to provide persistent storage for Elasticsearch running inside the EKS cluster.

### Why Do We Need EBS?

Elasticsearch stores a large amount of log data.

If the Elasticsearch pod restarts, we should not lose the stored data.

Persistent storage helps retain the data even when Kubernetes replaces the pod.

The teacher uses EBS volumes to store Elasticsearch data.

EBS snapshots can also be used to back up the stored data.

---

## 11. What is CSI Driver?

CSI stands for Container Storage Interface.

It is a standard interface that allows Kubernetes to communicate with storage systems through storage drivers.

In Amazon EKS, the Amazon EBS CSI Driver allows Kubernetes to provision and manage EBS volumes for workloads.

### Why Do We Need the EBS CSI Driver?

There is an important difference between the Kubernetes cluster and AWS storage.

- Elasticsearch runs inside the EKS cluster.
- EBS volumes are AWS storage resources.

Kubernetes needs a storage driver to provision and attach EBS volumes for the pods.

This is where the EBS CSI Driver comes in.

### Working Flow

```text
Elasticsearch Pod
       |
       v
PersistentVolumeClaim
       |
       v
StorageClass
       |
       v
EBS CSI Driver
       |
       v
AWS EBS Volume
       |
       v
Persistent Storage
```

The CSI driver allows Kubernetes to work with EBS storage.

---

## 12. What is IAM Role for Service Account (IRSA)?

IRSA stands for IAM Roles for Service Accounts.

It allows Kubernetes service accounts to be associated with AWS IAM roles.

In this practical, Elasticsearch needs to use EBS storage.

For this interaction, the Kubernetes service account used by Elasticsearch is associated with an IAM role that has the required AWS permissions.

### Why is IRSA Required?

Imagine Elasticsearch needs to create or manage an EBS volume.

The EKS workload needs permission to perform the required AWS operations.

Instead of storing AWS access keys inside the application or pod, I can associate a Kubernetes service account with an IAM role.

### Working Flow

```text
Elasticsearch Pod
        |
        v
Kubernetes Service Account
        |
        v
IAM Role
        |
        v
AWS Permissions
        |
        v
EBS Storage Operations
```

### Important Points

- IRSA connects Kubernetes service accounts with AWS IAM roles.
- It provides AWS permissions to workloads.
- It avoids storing long-term AWS access keys inside application containers.
- It is used in this practical to provide the required permissions for Elasticsearch storage operations.

---

## 13. What is Helm?

Helm is a package manager for Kubernetes.

It helps install, configure and manage Kubernetes applications using Helm charts.

Instead of manually creating many Kubernetes YAML files, I can use a Helm chart to deploy an application.

In this practical, the teacher uses Helm to install:

- Elasticsearch
- Kibana
- Fluent Bit

### Why Use Helm?

1. Simplifies application installation.
2. Reduces manual YAML configuration.
3. Allows configuration through values files.
4. Makes application upgrades and management easier.
5. Helps deploy complex Kubernetes applications.

### What is a Helm Chart?

A Helm chart is a collection of files that defines how a Kubernetes application should be installed.

A `values.yaml` file contains configuration values that can be customized during installation.

For example, in this practical, I need to update the Fluent Bit configuration with the Elasticsearch authentication details.

---

## 14. Understanding Fluent Bit Configuration

The teacher explains four important sections in the Fluent Bit configuration file.

They are:

1. Service
2. Input
3. Filter
4. Output

### 1. Service

The Service section contains general Fluent Bit service configuration.

It controls how Fluent Bit operates and which features are enabled.

### 2. Input

The Input section defines where Fluent Bit collects data from.

In this practical, Fluent Bit collects Kubernetes container logs.

The teacher uses the `tail` input plugin to read container log files.

### 3. Filter

The Filter section allows Fluent Bit to modify or filter the collected logs before forwarding them.

For example, the teacher demonstrates excluding logs from the `logging` namespace.

This helps avoid collecting unnecessary logs from the logging components themselves.

Fluent Bit also supports processing through Lua scripts.

### 4. Output

The Output section defines where Fluent Bit sends the collected logs.

In this practical, the output destination is Elasticsearch.

The configuration includes:

- Elasticsearch host.
- Elasticsearch port.
- Username.
- Password.
- TLS settings.

The teacher explains that TLS must be enabled when connecting to Elasticsearch in this AWS EKS setup.

### Complete Flow

```text
Fluent Bit Configuration
          |
    -----------------
    |       |       |
    v       v       v
 Service  Input   Filter
                    |
                    v
                  Output
                    |
                    v
              Elasticsearch
```

---

## 15. What is Kibana Query Language (KQL)?

KQL stands for Kibana Query Language.

It is used inside Kibana to search and filter log data.

For example, suppose I have logs from multiple Kubernetes namespaces.

I want to see only logs generated by the `dev` namespace.

I can use a KQL filter such as:

```text
kubernetes.namespace_name : "dev"
```

This helps me filter logs based on the namespace field when that field is available in the indexed log data.

KQL makes it easier to search for specific log messages and investigate application issues.

The teacher mentions KQL but does not cover it in depth in this episode.

---

## 16. Fluent Bit and Vendor Neutrality

One advantage of Fluent Bit is that it can work with different output destinations.

For example, I can use Fluent Bit to forward logs to Elasticsearch.

In the future, if an organization changes its log management platform, Fluent Bit can also be configured to forward logs to another supported destination.

This makes Fluent Bit useful in environments where the logging backend may change.

---

## 17. Important Interview Questions

### Q1. What is logging?

Logging is the process of recording messages and events generated by an application or system. These messages help developers understand application behaviour and troubleshoot problems.

### Q2. Why is logging important?

Logging helps identify errors, understand application behaviour, troubleshoot failures and investigate problems in distributed systems.

### Q3. What are the three pillars of observability?

The three pillars are Metrics, Logs and Traces.

### Q4. What is centralized logging?

Centralized logging collects logs from multiple applications and stores them in one central location so that users can search and investigate them from a single interface.

### Q5. What is EFK?

EFK is a centralized logging stack consisting of Elasticsearch, Fluent Bit and Kibana.

### Q6. What is the responsibility of Elasticsearch?

Elasticsearch stores, indexes and allows users to search application logs.

### Q7. What is Fluent Bit?

Fluent Bit is a lightweight log processor and forwarder that collects logs from different sources and sends them to a destination such as Elasticsearch.

### Q8. Why is Fluent Bit deployed as a DaemonSet?

A DaemonSet ensures that a Fluent Bit pod runs on every eligible Kubernetes node so that it can collect logs from each node.

### Q9. What is Kibana?

Kibana is a user interface used to search, explore and visualize log data stored in Elasticsearch.

### Q10. What is the difference between EFK and ELK?

Both stacks use Elasticsearch and Kibana. EFK uses Fluent Bit, while ELK uses Logstash. Fluent Bit is lightweight, whereas Logstash offers more extensive data processing capabilities.

### Q11. Why is persistent storage required for Elasticsearch?

Elasticsearch stores log data. Persistent storage helps retain this data even if the Elasticsearch pod restarts or is replaced.

### Q12. What is the EBS CSI Driver?

The EBS CSI Driver allows Kubernetes to provision and manage AWS EBS volumes for persistent storage.

### Q13. What is IRSA?

IRSA associates Kubernetes service accounts with AWS IAM roles so that workloads can securely access AWS services using assigned permissions.

### Q14. What are the four main sections of Fluent Bit configuration?

Service, Input, Filter and Output.

### Q15. What is KQL?

Kibana Query Language is used to search and filter log data inside Kibana.

---

## 18. Final Summary

In this theory session, I learned about the second pillar of observability: Logging.

I understood why logs are important for troubleshooting applications and why centralized logging is required in distributed systems.

I also learned about the EFK stack and its three main components:

- Elasticsearch stores and indexes logs.
- Fluent Bit collects and forwards logs.
- Kibana provides a user interface to search and visualize logs.

I understood how Fluent Bit runs as a DaemonSet on Kubernetes nodes and collects container logs.

I also learned the purpose of AWS EBS, the EBS CSI Driver, IRSA and Helm in setting up the logging stack on Amazon EKS.

### Complete EFK Architecture

```text
        Kubernetes Applications
                  |
                  v
             Container Logs
                  |
                  v
              Fluent Bit
             (DaemonSet)
                  |
                  v
             Elasticsearch
             (StatefulSet)
                  |
                  v
                Kibana
             (User Interface)
                  |
                  v
          Search and Troubleshooting
```

**My main takeaway:** Metrics help me understand the health and performance of an application, while logs help me investigate why an application is failing. Centralized logging makes this process easier by collecting logs from multiple applications in one place.

---

## Learning Source

**Teacher:** Abhishek Veeramalla

**Course:** Observability Zero to Hero – Episode 5

**Topic:** Logging and EFK Stack on Kubernetes

**GitHub Repository:**

https://github.com/iam-veeramalla/observability-zero-to-hero/tree/main/day-5

These notes are based on the teacher's Episode 5 explanation and the transcript provided for this lesson.
