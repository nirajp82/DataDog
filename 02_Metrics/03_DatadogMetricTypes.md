# Datadog Metric Types (Simple Explanation)

Regardless of where they come from, **all Datadog metrics fall into five main types**. Each type answers a different kind of question about your system’s behavior.

---

## 1. **Count**

**What it measures:**
The **total number of events** that happened during a time period.

**Think of it as:** A counter that keeps adding up.

**Examples:**

* Total HTTP requests made
* Number of application errors
* Number of database queries
* Number of service restarts

**Example metric names:**

* `http.requests.count`
* `app.errors.count`
* `db.query.count`

**Questions it helps answer:**

* How many errors happened in the last 2 hours?
* How many API calls were made today?
* How many transactions were processed yesterday?

---

## 2. **Rate**

**What it measures:**
How **frequently something is happening per second**.

It is calculated using:

> **Rate = Total Count ÷ Time (in seconds)**

**Think of it as:** Speed of events.

**Example:**
If 240 errors happened in 1 hour (3600 seconds):

* Error rate = `240 / 3600 = 0.066 errors per second`

**Examples:**

* HTTP requests per second
* Errors per second
* Database queries per second

**Example metric names:**

* `http.requests.rate`
* `app.errors.rate`
* `db.query.rate`

**Questions it helps answer:**

* Is traffic increasing or decreasing?
* Are errors happening faster than usual?
* Is request volume spiking?

---

## 3. **Gauge**

**What it measures:**
The **current value at a specific moment in time**.

**Think of it as:** A speedometer or thermometer.

It does **not add up over time**—it only shows the latest value.

**Examples:**

* Current CPU usage
* Memory in use right now
* Active users right now
* Current network bandwidth

**Example metric names:**

* `system.cpu.utilization`
* `system.memory.used`
* `app.users.active`

**Questions it helps answer:**

* How much CPU is being used right now?
* How much memory is currently in use?
* How many users are active at this moment?

---

## 4. **Histogram**

**What it measures:**
The **distribution of values over time**, grouped into ranges (buckets).

Instead of just showing an average, it shows **how values are spread out**.

**Think of it as:** Grouping values into ranges like:

* 0–50 ms
* 51–100 ms
* 101–150 ms

**Common use cases:**

* Request latency
* Query execution time
* Payload sizes
* Session duration

**What Datadog automatically calculates from histograms:**

* Average (mean)
* Median
* Count
* 95th percentile (p95)
* Maximum value

**Example metric names:**

* `http.request.latency`
* `db.query.execution_time`
* `user.session.duration`

**Questions it helps answer:**

* What is the typical request latency?
* How bad are the slowest requests?
* Are most users experiencing fast or slow performance?

---

## 5. **Distribution**

**What it measures:**
Similar to histograms, but **aggregated across many systems at once** (hosts, containers, regions).

**Key difference from Histogram:**

* **Histogram:** Per host or per service
* **Distribution:** Across the **entire environment**

This allows **global percentiles** like p95 and p99 across all servers.

**Example use cases:**

* API latency across all regions
* CPU usage across an entire cluster
* Page load time across all users worldwide

**Example metric names:**

* `api.request.latency`
* `web.page_load_time`
* `system.cpu.utilization`

**Questions it helps answer:**

* What is the global p99 latency this week?
* How does performance vary across regions?
* Are only a few servers slow or is the whole system slow?

---

## Quick Comparison Summary

| Metric Type      | What It Shows                                | Example Use            | Explanation                                                                                                                                                                                                                                                                                    |
| ---------------- | -------------------------------------------- | ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Count**        | Total events over a period                   | Total errors today     | Shows **how many times an event occurred**. For example, if your application had 452 errors today, the count metric tracks that total. It’s cumulative, just adding events over time.                                                                                                          |
| **Rate**         | Events per second                            | Requests per second    | Shows **how frequently an event is happening**. For example, if your API received 7,200 requests in 2 hours, the rate metric tells you it handled 1 request per second. It’s useful for monitoring traffic spikes or load patterns.                                                            |
| **Gauge**        | Current value at a moment                    | CPU usage right now    | Shows the **current state of a metric**, not cumulative. For example, a gauge could show your server is using 65% CPU at this exact moment. The value changes constantly as conditions change.                                                                                                 |
| **Histogram**    | Distribution of values (per host or service) | Request latency ranges | Shows **how values are spread across ranges** for a specific host/service. For example, if API request times range from 20ms to 250ms, a histogram groups them (0-50ms: 100 requests, 51-100ms: 80 requests, etc.) so you can see patterns like most requests being fast but a few being slow. |
| **Distribution** | Aggregated distribution across all systems   | Global p95 API latency | Shows **how values are spread across all hosts, containers, or regions**. For example, if multiple servers handle requests, the distribution metric can tell you the 95th percentile latency across all servers globally, helping identify system-wide performance bottlenecks.                |

---

## Why This Matters

Each metric type answers a **different kind of monitoring question**:

* **Count & Rate** help you understand **load and traffic**
* **Gauge** helps you understand **current system health**
* **Histogram & Distribution** help you understand **performance, variability, and worst-case behavior**

Using the wrong metric type can give **misleading results**, while using the correct one gives **accurate visibility and better alerts**.

---

If you want, I can also:

* Add **real Datadog dashboard examples**, or
* Create a **one-page cheat sheet PDF-style summary** for revision.
