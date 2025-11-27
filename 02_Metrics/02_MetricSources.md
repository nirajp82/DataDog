## Datadog Metric Sources and Collection Methods

Datadog can collect, aggregate, visualize, and alert on a very large variety of metrics. Metrics represent numeric measurements of your systems, applications, and business activity over time. 

---

## 1. Main Sources of Metrics in Datadog

Datadog metrics come from the following primary sources:

* **Datadog Agent**: The Datadog Agent is installed on a server or container to collect system-level metrics like CPU, memory, disk, and network. It sends this data directly to Datadog for monitoring.
* **Agent-based Integrations**: These are plugins that run with the Datadog Agent (on top of the Datadog Agent) to collect detailed metrics from services running on the same machine.They are plugins that allow the Agent to collect metrics from specific software running on the same machine, such as NGINX, MySQL, or Redis
* **DogStatsD (StatsD protocol)**
* **Datadog Integrations (Cloud, SaaS, Tools)**: These integrations collect metrics from cloud platforms and SaaS tools using APIs, such as AWS, Azure, Slack, and PagerDuty, not from machine (without needing the Datadog Agent installed).
* **Other Datadog Products**

  * Real User Monitoring (RUM)
  * Application Performance Monitoring (APM)
  * Logs
  * Processes
  * Events
* **Datadog API**

Each of these sources feeds numerical data into Datadog, where it can be stored, visualized, and alerting can be applied.

---

## 2. The Datadog Agent (Primary Metric Source)

The **Datadog Agent** is usually the first and most important source of metrics. It is a lightweight program installed on your:

* Servers
* Virtual machines
* Containers
* Kubernetes nodes

Once installed, it **automatically collects and sends standard system and service metrics** to Datadog.

### Host-Level Metrics Collected by the Agent

* CPU usage
* Memory usage
* Disk usage
* Disk I/O
* Network traffic

### Container Metrics

* Container CPU
* Container memory
* Container disk I/O
* Container network metrics

### APM Metrics (via the Agent)

* Service latency
* Request volume
* Error rates

### Network Metrics

* TCP connections
* DNS queries

These metrics are collected continuously and sent to Datadog without writing any custom code.

---

## 3. Core Integrations and DogStatsD

### Core Integrations

The Datadog Agent includes many **built-in (core) integrations**. These are officially developed and supported by Datadog and collect metrics from common tools such as:

* Web servers
* Databases
* Caches
* Operating systems

The Agent runs these integrations and sends their metrics automatically to Datadog.

---

### DogStatsD and the StatsD Protocol (Important for Custom Metrics)

Datadog bundles **DogStatsD** with the Agent. DogStatsD is Datadog’s implementation of the **StatsD protocol** and is mainly used to collect **custom application metrics**.

#### What is the StatsD Protocol?

StatsD is a **lightweight, text-based protocol** that applications use to send metrics in real time. Instead of Datadog pulling data, your application **pushes metrics** as simple text messages over UDP or TCP.

With StatsD, applications typically send:

* Counters (e.g., number of requests)
* Gauges (e.g., current CPU usage)
* Timers (e.g., request latency)
* Histograms and distributions

A StatsD server receives these metrics, aggregates them, and forwards them to a monitoring backend such as Datadog.

#### What is DogStatsD?

DogStatsD is Datadog’s enhanced version of StatsD. In addition to standard StatsD features, it supports:

* Tags
* Service context
* Distributions
* Datadog-specific metric enhancements

DogStatsD is **the most common way to send custom business and application metrics** to Datadog, such as:

* User signups
* Checkout attempts
* Payment failures
* Feature usage

---

## 4. Datadog Integrations (Beyond the Agent)

Datadog provides **800+ ready-made integrations** for cloud platforms and third-party tools such as:

* AWS
* Azure
* Slack
* PagerDuty
* Databases, queues, and messaging systems

### Types of Datadog Integrations

1. **Agent-Based Integrations**
   Installed and run by the Datadog Agent on your host.

2. **API-Based (Authentication / Crawler) Integrations**
   Datadog connects to third-party services using API credentials and pulls metrics directly.

3. **Library Integrations**
   Added into application code (Node.js, Python, etc.) to send application-level telemetry.

---

## 5. Metrics from Other Datadog Products

Datadog can **generate metrics from data already collected** by other parts of the platform.

### From Logs

You can turn logs into metrics such as:

* Number of HTTP requests
* Total API calls
* Count of 500-level errors
* Percentage of successful responses

### From APM (Application Performance Monitoring)

You can generate metrics from traces and spans, including:

* Total requests
* Average request duration
* Transactions processed
* Throughput per service

### From RUM (Real User Monitoring)

You can generate user-experience metrics such as:

* Total user sessions
* Page load time
* Average session duration
* Performance by browser type

---

## 6. Custom Metrics

Even with Agents, integrations, logs, APM, and RUM, you often need **business-specific metrics**, such as:

* KPIs
* Custom algorithms
* User registrations
* Feature usage
* Revenue-related events

These are called **custom metrics**. They are not collected automatically—you write code to generate and send them.

### Ways to Send Custom Metrics to Datadog

* **Datadog API**
  Use the HTTP API to submit metrics directly.

* **DogStatsD (Most Common)**
  Applications send metrics to DogStatsD, which forwards them through the Agent.

* **Custom Agent Checks**
  Write your own check that runs inside the Datadog Agent.

* **Integrations**

  * Some standard integrations can emit custom metrics.
  * Some Marketplace integrations also support custom metrics.

---

## 7. What’s Next

Now that you understand:

* Where Datadog metrics come from
* How the Agent, integrations, DogStatsD, and APIs work together

The next step is to learn about:

* **Metric types**
* **Metric formats**
* **How metrics are queried and visualized**

After that, you will be ready for **hands-on labs** where you will explore, query, and visualize metrics directly in Datadog.
