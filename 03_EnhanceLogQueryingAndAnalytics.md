## **Enhance Log Querying and Analytics with Reference Tables, Subqueries, and Calculated Fields**

When working with logs, you’ll often need more than basic searching and aggregations. Sometimes you need to enrich logs with external data, connect one set of logs to another, or generate new values that don’t already exist in the logs. Datadog provides three powerful features that make this possible: **Reference Tables**, **Subqueries**, and **Calculated Fields**. Together, these tools help you analyze logs with deeper context, build smarter queries, and uncover insights that aren’t visible from raw log data alone.

### **1. Reference Tables**

Reference Tables let you upload external data—such as CSV files, lookup lists, or metadata—and join it with your logs. This allows you to enrich logs with additional context.

**Example:**
Upload a table of product names and product IDs, then join it with your logs so that product ID fields automatically display the product name.

---

### **2. Subqueries**

Subqueries allow you to use the result of one log search to filter or refine another search. This is useful when two types of logs are related and you want to connect their data.

**Example:**
Find customer IDs involved in checkout failures, then use those IDs to search through webserver logs to find what happened before the failure.

---

### **3. Calculated Fields**

Calculated Fields let you create new fields based on existing tags or attributes in your logs. This is useful for generating metrics or values that aren’t originally recorded.

**Example:**
Compute a new field called “profit” by subtracting cost from revenue, even if profit is not directly logged.

---

These features make your log analysis more flexible and powerful, enabling deeper insights and more meaningful visualizations as you work with Datadog Log Explorer.
