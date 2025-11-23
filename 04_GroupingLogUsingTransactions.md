## **What is a Transaction?**

**Transactions** group together logs that are part of the same sequence of events, such as a user session or a request that passes through multiple services.

Unlike normal grouping, transaction grouping includes not just the logs that match your search, but **all logs that are part of the related sequence**, giving you a complete picture of the activity.

**Example:**
A customer browsing an e-commerce website might create multiple logs:

* Search for a product
* View product details
* Add product to cart
* Checkout

All these logs share a common **orderId** or **requestId**. Datadog can group them into a **transaction** to view the entire customer activity as one unit.

---

## **Why are Transactions different from normal "Group By"?**

* **Group By:** Only groups logs that **match your search query**.
  <img width="1761" height="403" alt="image" src="https://github.com/user-attachments/assets/79a99e9c-fb61-4f4f-a329-a6b42d2d12f0" />
  <img width="1818" height="529" alt="image" src="https://github.com/user-attachments/assets/6640e197-a1e9-4d98-99ba-765696e2bc54" />

* **Transactions:** Groups **all logs related to the activity**, even if some logs do not match the query.
  <img width="1787" height="413" alt="image" src="https://github.com/user-attachments/assets/938d6506-b77a-4911-be5c-6d31aff2e44a" />
  <img width="1808" height="933" alt="image" src="https://github.com/user-attachments/assets/0d7f7fbe-fe30-4263-b991-d19600773999" />
### **Example:**

You search for logs with the keyword **"ERROR"**.

| Log # | order_id | message                 |
| ----- | -------- | ----------------------- |
| 1     | 123      | User searched for shoes |
| 2     | 123      | Added to cart           |
| 3     | 123      | Checkout started        |
| 4     | 123      | ERROR — payment failed  |
| 5     | 123      | Retry payment           |
| 6     | 123      | Payment success         |

* **Group By order_id** (matching logs only):

| order_id | count of error logs |
| -------- | ------------------- |
| 123      | 1                   |

* **Transaction (grouped by order_id)**:

| Log # | order_id | message                 |
| ----- | -------- | ----------------------- |
| 1     | 123      | User searched for shoes |
| 2     | 123      | Added to cart           |
| 3     | 123      | Checkout started        |
| 4     | 123      | ERROR — payment failed  |
| 5     | 123      | Retry payment           |
| 6     | 123      | Payment success         |

Transactions show **all related logs**, giving you the full context.

---

## **What can you measure in Transactions?**

### **1. Duration**

Time between the first and last log in a transaction.

### **2. Maximum Severity**

The highest severity found in any log within the transaction.

### **3. Key Items**

For any string field, you can calculate:

* Count unique values
* Latest value
* Earliest value
* Most frequent value

### **4. Statistics for numerical fields**

Compute statistics like:

* min, max, avg, sum, median
* Percentiles: p75, p90, p95, p99

### **5. Start and End Conditions**

You can define **custom transaction boundaries**:

* **Start condition:** e.g., "checkout-start" log
* **End condition:** e.g., "checkout-complete" log

This helps include only relevant logs for each transaction.

---

## **Example Use Case**

**Scenario:** E-commerce checkout process

Logs for a single order (`order_id = 987`):

1. Search for shoes
2. View product
3. Add to cart
4. Checkout
5. Payment failed
6. Retry payment
7. Payment success

**Transaction view:**

* Duration: 12.4 seconds
* Maximum severity: ERROR (from payment failed)
* Count unique products viewed: 1
* Total cart price: $85
* All 7 logs are included

This gives you the **complete picture of what happened** for one order, not just the error.

---

## **Why use Transactions?**

* Debug multi-step workflows easily
* Trace activity across multiple microservices
* Understand user behavior from start to finish
* Measure performance and duration of key activities
* Detect patterns in failures or errors

---

**Summary:**
Transactions in Datadog let you group related logs together, measure key metrics, and get the full context for each activity. Unlike normal Group By aggregations, transactions include all logs in the activity, even if they do not match your query, making it easier to analyze complex workflows.
