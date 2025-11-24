# Enhance Log Querying and Analytics with Reference Tables, Subqueries, and Calculated Fields

When working with logs, you often need more than simple searching and aggregations. Sometimes you need to enrich logs with external data, connect separate log streams, or generate new values that don’t already exist in your logs. Datadog provides three powerful features that enable these deeper insights: **Reference Tables**, **Subqueries**, and **Calculated Fields**.

These tools expand how you analyze and understand log data, making it possible to create richer queries, build meaningful visualizations, and derive insights that raw logs alone cannot provide.
---
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

    - Q3: Using the flagged_ip_addresses reference table and the service:webserver logs, create a bar chart that compares the number of flagged and not-flagged IP addresses seen in the service.
      <img width="1763" height="749" alt="image" src="https://github.com/user-attachments/assets/e9bae472-a89a-40a5-8d20-c35ab8274d4e" />
     
## Subqueries

Subqueries let you use the result of one log search to filter or refine another search. This is useful when working with related log sources.

**Use case:** Connect and analyze relationships between two different sets of logs.

**Example:**
- Q: Create a timeseries graph for the percentage of products without a discount applied at checkout. Display the data as a line.
   <img width="1769" height="869" alt="image" src="https://github.com/user-attachments/assets/94535db8-ff38-4903-bbe4-3cc5adeab74e" />

   - Showing all the data
   <img width="1767" height="842" alt="image" src="https://github.com/user-attachments/assets/0280d754-a863-48e8-8815-2ef3fd925949" />

-   When a cart checkout fails, a `service:cart-logger` log is created. Some customers may attempt checkout again, while others may cancel the checkout. Even if a customer cancels, all related checkout logs are       still generated, but the `service:cart-checkout` log will show the `checkout_status` as **ABANDONED**.

    - Q: **analyze how failed checkouts impact overall sales** by determining what portion of these carts are eventually **ABANDONED** versus **SUCCEEDED**.

      Create a visualization (e.g., a Pie Chart) to show the **percentages of carts with `checkout_status` ABANDONED and SUCCEEDED**. This will help understand customer behavior after failed checkout attempts.
      <img width="1126" height="554" alt="image" src="https://github.com/user-attachments/assets/4f68a33b-92a0-4610-a943-3cbb26a4751e" />

    - Q: Create a timeseries for the `Cumulative Sum` (in the Arithmetic catergory) of cart totals for abandoned carts that results from failed checkouts over a period of 30 minutes.  
      <img width="1759" height="616" alt="image" src="https://github.com/user-attachments/assets/651e26e7-dc02-458a-863b-0ff31f0f1b23" />

---

## Calculated Fields

Calculated Fields in Datadog let you create temporary, custom fields from existing log data while working in Log Explorer. These fields behave like normal log attributes, which means you can use them for searching, aggregating, visualizing, or even building other calculated fields.

Calculated fields are temporary, visible only to you, and ideal for retroactive analysis because they can be applied to logs that have already been indexed. You reference them with a # prefix in queries, aggregations, or other calculations, and you can create up to five calculated fields at a time.

### Types of Calculated Fields

1. **Formula**

   * Use formulas to create new values from existing attributes.
   * You can perform arithmetic, manipulate text, or use conditional logic.
   * Example: `#latency_gap = @client_latency - @server_latency`

2. **Extraction**

   * Use Grok parsing rules to extract values from raw log messages or attributes.
   * Works retroactively on indexed logs without changing pipelines.
   * For example, given a log line like `country=Brazil duration=123ms path=/index.html status=200 OK`, you can use a Grok extraction rule such as `country=%{WORD:country} duration=%{INTEGER:duration} path=%{NOTSPACE:request_path} status=%{DATA:status}` to create calculated fields on the fly. This produces `#country = Brazil`, `#duration = 123`, `#request_path = /index.html`, and `#status = 200 OK`, allowing you to analyze log data without modifying your ingestion pipeline.

