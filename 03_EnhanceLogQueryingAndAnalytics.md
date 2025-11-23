# Enhance Log Querying and Analytics with Reference Tables, Subqueries, and Calculated Fields

When working with logs, you often need more than simple searching and aggregations. Sometimes you need to enrich logs with external data, connect separate log streams, or generate new values that don’t already exist in your logs. Datadog provides three powerful features that enable these deeper insights: **Reference Tables**, **Subqueries**, and **Calculated Fields**.

These tools expand how you analyze and understand log data, making it possible to create richer queries, build meaningful visualizations, and derive insights that raw logs alone cannot provide.

## Reference Tables

Reference Tables allow you to upload external data—such as CSV files, metadata lists, or lookup tables—and join that information with your logs.

**Use case:** Enrich logs with additional context that isn’t originally included.

**Example:**
Upload a table containing product names and product IDs. Joining this table with your logs lets you see product names whenever a product ID appears in log entries.

---

## Subqueries

Subqueries let you use the result of one log search to filter or refine another search. This is useful when working with related log sources.

**Use case:** Connect and analyze relationships between two different sets of logs.

**Example:**
Identify customer IDs associated with checkout failures, then use those same customer IDs to search through webserver access logs to understand what led to the failure.

---

## Calculated Fields

Calculated Fields let you generate new values based on existing attributes in your logs.

**Use case:** Create metrics or computed values not originally logged.

**Example:**
If revenue and cost are logged separately, you can create a new field called **profit** by calculating `revenue - cost`, even though the profit field never existed in the original logs.

---

These three features—Reference Tables, Subqueries, and Calculated Fields—extend what you can do inside Log Explorer, enabling more insightful queries, flexible analysis, and enhanced visualizations.
