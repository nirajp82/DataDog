## What is a Transaction?

A **transaction** groups together logs that are all part of the same sequence of events, such as a user session or a request moving across multiple services. Unlike a standard group-by, transactions include **all logs that belong to the related activity**, not just those matching your search. This gives you a complete view of what happened.

**Example:**
A customer on an e-commerce site might generate these logs:

* Search for a product
* View product details
* Add product to cart
* Checkout

All these logs share a common **orderId** or **requestId**, so Datadog can group them into a single transaction for analysis.

**Realistic Scenario:**

* `service:cart-checkout` with `@customer.id:109` and `@checkout_status:ABANDONED`
* Logs captured:

  1. Added item to cart
  2. Updated item quantity
  3. Viewed cart summary
  4. Checkout abandoned
* All these logs form one transaction showing the full journey of the abandoned checkout.

---

## **Why are Transactions different from normal "Group By"?**

* **Group By:** Only groups logs that **match your search query**.
  <img width="1761" height="403" alt="image" src="https://github.com/user-attachments/assets/79a99e9c-fb61-4f4f-a329-a6b42d2d12f0" />
  <img width="1818" height="529" alt="image" src="https://github.com/user-attachments/assets/6640e197-a1e9-4d98-99ba-765696e2bc54" />

* **Transactions:** Groups **all logs related to the activity**, even if some logs do not match the query.
  <img width="1787" height="413" alt="image" src="https://github.com/user-attachments/assets/938d6506-b77a-4911-be5c-6d31aff2e44a" />
  <img width="1808" height="933" alt="image" src="https://github.com/user-attachments/assets/0d7f7fbe-fe30-4263-b991-d19600773999" />
### **Example:**

Logs:

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
  |----------|-------------------|
  | 123      | 1                 |

* **Transaction (grouped by order_id):**
  | Log # | order_id | message                 |
  |-------|----------|-------------------------|
  | 1     | 123      | User searched for shoes |
  | 2     | 123      | Added to cart           |
  | 3     | 123      | Checkout started        |
  | 4     | 123      | ERROR — payment failed  |
  | 5     | 123      | Retry payment           |
  | 6     | 123      | Payment success         |

Transactions give the **full context** for the activity.

---

## What Can You Measure in Transactions?

### 1. Duration

Time between the first and last log in the transaction.

### 2. Maximum Severity

Highest severity found in any log within the transaction.

### 3. Key Items

For string fields, calculate:

* Count unique values
* Latest value
* Earliest value
* Most frequent value

### 4. Statistics for Numbers

Compute statistics like:

* min, max, avg, sum, median
* Percentiles: p75, p90, p95, p99

### 5. Start and End Conditions

Define custom transaction boundaries:

* **Start condition:** e.g., "checkout-start"
* **End condition:** e.g., "checkout-complete"
  This ensures only relevant logs are included.

---

## Example Use Case: E-commerce Checkout

Logs for `order_id = 987`:

1. Search for shoes
2. View product
3. Add to cart
4. Checkout
5. Payment failed
6. Retry payment
7. Payment success

**Transaction View:**

* Duration: 12.4 seconds
* Maximum severity: ERROR
* Count unique products viewed: 1
* Total cart price: $85
* All 7 logs included

This shows the **complete customer journey**, not just the error.

---

## Why Use Transactions?

* Debug multi-step workflows
* Trace activity across multiple services
* Understand user behavior from start to finish
* Measure performance and duration
* Detect patterns in failures or errors

---

**Summary:**
Transactions let you group related logs into a single, coherent view. Unlike normal Group By aggregations, they include all logs in the activity, giving you **full context** for better debugging, analysis, and visualization.

