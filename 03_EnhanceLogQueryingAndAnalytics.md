# Enhance Log Querying and Analytics with Reference Tables, Subqueries, and Calculated Fields

When working with logs, you often need more than simple searching and aggregations. Sometimes you need to enrich logs with external data, connect separate log streams, or generate new values that don’t already exist in your logs. Datadog provides three powerful features that enable these deeper insights: **Reference Tables**, **Subqueries**, and **Calculated Fields**.

These tools expand how you analyze and understand log data, making it possible to create richer queries, build meaningful visualizations, and derive insights that raw logs alone cannot provide.

## Reference Tables

Reference Tables allow you to upload external data—such as CSV files, metadata lists, or lookup tables—and join that information with your logs.

**Use case:** Enrich logs with additional context that isn’t originally included.

**Example:**
Upload a CSV containing flagged IP addresses with the columns id and ip_address. Joining this table with your logs lets you easily identify when a log entry contains an IP address that appears in the flagged list.

<img width="1566" height="404" alt="image" src="https://github.com/user-attachments/assets/4fca6bfa-7c95-43f7-8580-3284290963a8" />

<img width="1534" height="497" alt="image" src="https://github.com/user-attachments/assets/04e63044-e989-4076-8b08-5958448306ce" />

In the example below, IP address data from the flagged_ip_addresses reference table is being used to search all recent service:webserver logs for malicious IP addresses. The service:webserver logs are joined with the flagged_ip_addresses table where the Client IP (@network.client.ip) attribute in the logs is in the ip_address column in the table. There are a few events that match this search.

<img width="1520" height="755" alt="image" src="https://github.com/user-attachments/assets/75f5fc62-bea7-4de1-b3c4-88ef177bd3b0" />

A corresponding SQL query would be as follows. (The syntax may not be exact.)

```sql
SELECT *
FROM service:webserver
JOIN flagged_ip_addresses ON Client IP = ip_address
```
---

- Example 2: Search for `service:cart-checkout` Inner join with reference table customer_list where field Customer ID IN column id and show columns name

  <img width="1039" height="600" alt="image" src="https://github.com/user-attachments/assets/66902c66-a510-4ad2-807f-230848a5108b" />

  - Q1: Create a Top List of the top 10 sum of Cart Total by name. Which customer has the third highest cart total?

    <img width="1760" height="504" alt="image" src="https://github.com/user-attachments/assets/99a1d6ce-77fa-48d5-9aa6-85164246a205" />

  - Q2: Create a scatter plot showing the top 10 countries based on the total cart value (sum of Cart Total) plotted against the total number of items added to the cart (sum of Cart Item Quantity).
After creating the chart, determine which country has the lowest total cart value, and calculate its ratio of Cart Total to Cart Item Quantity.

    <img width="1755" height="803" alt="image" src="https://github.com/user-attachments/assets/43fdf7ea-0ad9-44e5-bbd4-84c244dbc534" />

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
