# Datadog Metrics

## Overview

To understand any system, you need to measure it. Just like we track heart rate for the body or temperature for a machine, digital systems are monitored using **metrics**. A single measurement shows only the current state, but when measurements are collected continuously and aggregated over time, they reveal trends, patterns, and system health.

## Why Aggregation Matters

One data point is limited, but aggregated data helps answer important questions such as:

* What is normal vs abnormal behavior?
* Is the system improving or degrading?
* Are values trending up or down?
* What patterns repeat over time?
  This allows you to detect issues early and take preventive action before failures occur.

## Metrics in Datadog

In Datadog, **metrics are numerical values collected over time** that represent the behavior and performance of systems. Examples include:

* CPU and memory usage
* Latency and request duration
* Error rates
* Business metrics like user sign-ups or transactions

Metrics help you monitor performance, detect anomalies, and forecast future behavior.

## Metric Format

Each individual metric data point contains:

* A **timestamp**
* A **numeric value**

Example:
`12 Dec 2024 22:42:51 – 65.63`

The unit (such as millicores for CPU) is stored in the metric’s metadata, not in every data point.

## Visualizing Metrics

When metrics are aggregated over time, they are displayed as **time series graphs**. These graphs show:

* **X-axis:** Time
* **Y-axis:** Metric value (with unit)

From these graphs, you can identify spikes, trends, seasonal patterns, and abnormal behavior.

## Why Metrics Are Important

By analyzing metrics over time, you can:

* Understand system performance
* Correlate metrics (e.g., CPU vs latency)
* Detect performance degradation early
* Identify what happens before outages or failures
* Make informed decisions to prevent incidents before they grow larger
