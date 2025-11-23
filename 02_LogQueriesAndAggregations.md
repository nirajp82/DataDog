# DataDog Log Queries & Aggregations

This document provides a reference for searching, filtering, and aggregating logs in DataDog using an e-commerce application dataset. It includes **log search syntax**, **log schemas**, and practical query examples.

---

## 1️⃣ Log Search Syntax (Refresher)

| Element               | Syntax                     | Example                                                                                        | Notes                                   |
| --------------------- | -------------------------- | ---------------------------------------------------------------------------------------------- | --------------------------------------- |
| **Tag**               | `key:value`                | `service:frontend`                                                                             | Filter logs by tag value.               |
| **Attribute**         | `@key:value`               | `@http.method:POST`                                                                            | Filter logs by log attribute.           |
| **Single term**       | `<text>`                   | `ResponseString`                                                                               | Searches log message text.              |
| **Exact phrase**      | `"search text"`            | `"Response fetched"`                                                                           | Searches for the exact string.          |
| **Conditional logic** | `AND`, `OR`, `-`           | `authentication AND password`<br>`service:(discounts OR ads)`<br>`service:frontend -status:OK` | Combine conditions; `-` negates a term. |
| **Wildcard**          | `*`                        | `DISCOUNT1*`<br>`service:store-*`                                                              | Matches multiple characters.            |
| **Numeric / range**   | `<, >, <=, >=, [## TO ##]` | `@http.response_time:>100`<br>`@http.status_code:[400 TO 499]`                                 | Filters numeric fields or ranges.       |

---

## 2️⃣ Lab Data Schema

Logs are from **five services** of an e-commerce application.  ** Every log contains `cart_id` and `customer.id`. **

---

### 2.1 Cart Checkout Logs

Logs created when a cart is **checked out** or **abandoned**.

| Key             | Type    | Description                            |
| --------------- | ------- | -------------------------------------- |
| cart_id         | string  | Unique cart identifier                 |
| customer.id     | string  | Customer ID                            |
| checkout_status | string  | Status of checkout (success/abandoned) |
| start_time      | double  | Checkout start timestamp               |
| item_quantity   | integer | Number of items in cart                |
| cart_total      | double  | Total cart value                       |
| checkout_time   | double  | Checkout completion timestamp          |
| message         | string  | Status message                         |
| status          | string  | Checkout status                        |
| service         | string  | Service name                           |
| ddsource        | string  | DataDog source                         |
| version         | string  | Service version                        |
| env             | string  | Environment                            |

**Query Example:**

```text
service:cart-checkout AND @checkout_status:success
```

---

### 2.2 Product Checkout Logs

Logs generated for **each product in a cart**.

| Key                 | Type    | Description           |
| ------------------- | ------- | --------------------- |
| cart_id             | string  | Cart identifier       |
| customer.id         | string  | Customer ID           |
| product.id          | string  | Product identifier    |
| product.url         | string  | Product URL           |
| product.unit_price  | double  | Price per unit        |
| quantity            | integer | Quantity purchased    |
| discount.code       | string  | Discount code applied |
| discount.percentage | double  | Discount percentage   |
| subtotal            | double  | Price after discount  |
| total               | double  | Total price           |
| checkout_time       | double  | Checkout timestamp    |
| message             | string  | Log message           |
| service             | string  | Service name          |
| ddsource            | string  | DataDog source        |
| version             | string  | Service version       |
| env                 | string  | Environment           |

**Query Example:**

```text
service:product-checkout AND @discount.code:DISCOUNT10
```

---

### 2.3 Discount Service Logs

Logs track **automatic discounts** applied during checkout.

| Key           | Type   | Description           |
| ------------- | ------ | --------------------- |
| discount_code | string | Applied discount code |
| cart_id       | string | Cart identifier       |
| customer.id   | string | Customer ID           |
| message       | string | Log message           |
| service       | string | Service name          |
| ddsource      | string | DataDog source        |
| version       | string | Service version       |
| env           | string | Environment           |

**Query Example:**

```text
service:discount-service AND @discount_code:BLACKFRIDAY
```

---

### 2.4 Web Server Logs

Logs generated when a **checkout is attempted**.

| Key                   | Type    | Description                     |
| --------------------- | ------- | ------------------------------- |
| cart_id               | string  | Cart ID                         |
| customer.id           | string  | Customer ID                     |
| date_access           | double  | Timestamp of request            |
| network.client.ip     | string  | Client IP                       |
| network.bytes_written | integer | Bytes sent to client            |
| http.method           | string  | HTTP method                     |
| http.url              | string  | Request URL                     |
| http.useragent        | string  | Client User-Agent               |
| http.status_code      | integer | HTTP response code              |
| http.status_category  | string  | Response category (2xx/4xx/5xx) |
| service               | string  | Service name                    |
| ddsource              | string  | DataDog source                  |
| version               | string  | Service version                 |
| env                   | string  | Environment                     |

**Query Example:**

```text
service:webserver AND @http.status_code:[500 TO 599]
```

---

### 2.5 Cart Logger Logs

Logs generated when a **checkout fails**.

| Key             | Type   | Description         |
| --------------- | ------ | ------------------- |
| cart_id         | string | Cart ID             |
| customer.id     | string | Customer ID         |
| cart_url        | string | Cart URL            |
| checkout_status | string | Failed or abandoned |
| checkout_time   | double | Checkout timestamp  |
| message         | string | Error message       |
| status          | string | Checkout status     |
| service         | string | Service name        |
| ddsource        | string | DataDog source      |
| version         | string | Service version     |
| env             | string | Environment         |

**Query Example:**

```text
service:cart-logger AND @checkout_status:failed
```

## Exercise

- Create a timeseries chart displaying the count of unique(`Count unique of`) `cart_id` for `service:cart-checkout`, using a 10-second rollup to aggregate and show the number of cart checkouts in each interval.
  <img width="1906" height="896" alt="image" src="https://github.com/user-attachments/assets/8298055b-ec23-48d1-aab5-2876b0f5b8b0" />

- Create a Top List showing the sum of `cart_total` for carts with `@item_quantity` between 3 and 5 Aggregate by `@item_quantity` and display only the top 2 values. This chart highlights the relationship between the number of items in a cart and the total cart value. 
  <img width="1898" height="445" alt="image" src="https://github.com/user-attachments/assets/6667043d-ecf3-4ac7-a7d3-4a02e1c376a2" />
  
  <img width="1909" height="477" alt="image" src="https://github.com/user-attachments/assets/e4cae831-7c95-4e98-97a1-ad76a72da634" />

- Find the minimum number of items in a cart required to reach a cart_total of 500 or more
  <img width="1908" height="723" alt="image" src="https://github.com/user-attachments/assets/dc430cbe-a1c4-4243-ba9e-997bdc2d5881" />


### References
https://learn.datadoghq.com/courses/enhance-log-querying
