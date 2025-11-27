## Metric Sources in Datadog (with StatsD Explained)

Datadog can collect, aggregate, visualize, and alert on metrics from many different sources. These metrics come from agents, integrations, applications, logs, APIs, and other Datadog products. Together, they give you full visibility into your infrastructure, applications, and business performance.

---

### The Datadog Agent

The **Datadog Agent** is the primary and most common source of metrics. It runs on your hosts, virtual machines, or containers and automatically collects system-level data such as:

* **Host metrics:** CPU usage, memory usage, disk usage, disk I/O, network traffic
* **Container metrics:** Container CPU, memory, disk I/O, and network usage
* **APM metrics:** Service latency, request counts, error rates
* **Network metrics:** TCP connections, DNS queries

Once installed, the Agent continuously sends these metrics to Datadog without any manual coding.

---

### Core Integrations and DogStatsD (StatsD Explained)

The Datadog Agent includes many **core integrations** that collect metrics from common tools like web servers, databases, and operating systems.

The Agent also includes **DogStatsD**, which is Datadog’s implementation of the **StatsD protocol**.

#### What is StatsD?

**StatsD** is a lightweight protocol used by applications to send real-time metrics. Instead of waiting for the Agent to “pull” data, your application **actively pushes metrics** to Datadog using simple text messages over UDP or TCP.

With StatsD, an application can easily send:

* Counters (e.g., number of requests)
* Gauges (e.g., current memory usage)
* Timers (e.g., request latency)
* Histograms and distributions

#### What is DogStatsD?

**DogStatsD** is Datadog’s enhanced version of StatsD. It:

* Supports all standard StatsD metric types
* Adds **Datadog-specific features** like tags, service context, and distributions
* Sends the aggregated metrics directly to Datadog through the Agent

This makes DogStatsD the most common way to send **custom application metrics** to Datadog. For example, your application can report:

* Number of user logins
* Checkout success/failure counts
* Payment processing time
* Business KPIs

---

### Datadog Integrations

Datadog provides **800+ ready-made integrations** for platforms such as AWS, Azure, databases, messaging systems, and SaaS tools. There are three main integration types:

* **Agent-based integrations:** Installed with the Datadog Agent
* **API-based (crawler) integrations:** Datadog pulls metrics using your credentials
* **Library integrations:** Added directly into your application code (Node.js, Python, etc.)

These integrations allow you to collect metrics without building custom pipelines.

---

### Metrics from Other Datadog Tools

Datadog can also generate metrics from data collected by other Datadog products:

* **Logs:** HTTP request counts, error rates, response success percentages
* **APM:** Throughput, average latency, total requests per service
* **RUM:** User sessions, page load time, session duration, browser performance

This allows you to turn raw logs and traces into meaningful numerical metrics.

---

### Custom Metrics

Sometimes your business needs metrics that Datadog does not collect by default, such as:

* Business KPIs
* User registrations or logins
* Feature usage
* Custom performance indicators

These **custom metrics** can be sent to Datadog using:

* **DogStatsD (most common)**
* **Datadog HTTP API**
* **Custom Agent checks**
* **Certain standard and Marketplace integrations**

---

## Summary

* The **Datadog Agent** collects system and service metrics automatically.
* **DogStatsD (StatsD)** lets your applications push **custom real-time metrics** to Datadog.
* **Integrations** collect metrics from cloud services and tools.
* **Logs, APM, and RUM** can also generate metrics.
* **Custom metrics** let you track business-specific data.

---

## What’s Next

Now that you understand where metrics come from and how StatsD fits in, the next step is learning about **metric types and formats**, followed by hands-on querying and visualization in Datadog.
