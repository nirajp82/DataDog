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

- Count how many unique carts are checked out in each 10-second interval and display it as a timeseries chart. (`Count unique of`)
  - Ori. Question: Select Timeseries for Visualize as. Update the Display type for the Timeseries to Lines. Then, build the following query:

    Search for service:cart-checkout

    Show Count unique of Cart ID by Everything rollup every 10s

    The rollup interval is the interval of time your data is aggregated over to create a datapoint in the chart. In this case, you're viewing the number of cart checkouts for every 10 seconds.

    <img width="1906" height="896" alt="image" src="https://github.com/user-attachments/assets/8298055b-ec23-48d1-aab5-2876b0f5b8b0" />

- Find all carts between 3 to 5 items and determine which carts, based on their item quantity (Cart with total item 3, 4 or 5), have the highest total cart value.
  - Ori. Question: Build the following query to create a Top List of the sum of cart totals.

    Search for @item_quantity:[3 TO 5]

    Show Sum of Cart Total by @item_quantity limit top 2

    This top list shows the relationship between item quantity in a cart and the cart total (price). Note that only 2 bars are displayed because you selected top 2. You can change the value to top 5 or to bottom 2 to see how the chart display changes.

    <img width="1898" height="445" alt="image" src="https://github.com/user-attachments/assets/6667043d-ecf3-4ac7-a7d3-4a02e1c376a2" />
  
    <img width="1909" height="477" alt="image" src="https://github.com/user-attachments/assets/e4cae831-7c95-4e98-97a1-ad76a72da634" />

- Find the minimum number of items in a cart required to reach a cart_total of 500 or more
  - Ori. Question: Build a query to answer the following question:

    What is the minimum number of items in a cart needed for a cart total of greater than or equal to 500? Display the data as a Timeseries chart with Bars.

    Search for @cart_total:>=500

    Show Sum of Cart Total by @item_quantity limit to top 25 rollup every 5s

    <img width="1908" height="723" alt="image" src="https://github.com/user-attachments/assets/dc430cbe-a1c4-4243-ba9e-997bdc2d5881" />

    Ori. Answer: A minimum of 3 or 4 items in a cart are needed for a cart total of greater than or equal to 500.

- Determine which 5 products people typically add in the largest quantities when they place them in their cart. After identifying these top 5 products, change the sorting method so that instead of ranking them by average quantity per cart, they are ranked by total quantity added. Observe how the top 5 products change when switching from average to total.
  - Ori. Question: Next, you're going to use the advanced options for limit to to sort the data displayed.

    - Build the following query to construct a Tree Map:

      Search for service:product-checkout

      Show Avg of Product Quantity by Product ID limit top 5

      The tree map displays the Product IDs with the top 5 average Product Quantity (the total number of unit items of a product in cart checkouts).

    - To adjust statistics for the Product Quantity that is used to sort the data, click the three vertical dots on the right of top 5. The default is sorted by Avg Product Unit Price. Update Avg to Sum to see how the sort order can affect the data that is displayed.

  <img width="1905" height="740" alt="image" src="https://github.com/user-attachments/assets/54ef7741-b594-4275-a20d-3c8ba916a8d5" />

- Ori. Question: To build the next query, you're going to use wildcards in the search term and display the statistics for multiple attributes in a table.

  Create a table for top 10 products with product IDs that start with the digits 1 or 44. The table should display the average of Product Unit Price, the sum of Product Quantity, and the sum of Product Total. The statistics should be listed by Product ID.

  Search for @product.id:(1* OR 44*)

  Show Sum of Product Quantity and Sum of Product Subtotal and Sum of Product Total by Product ID limit to top 10 sorted by Sum of Product Total

  <img width="1775" height="668" alt="image" src="https://github.com/user-attachments/assets/c04e4892-9314-4be5-aea0-2c85d1f8dd4c" />

## Display hierarchical group by aggregations
  - Fields aggregations support one group by dimension for the Top List visualization, and up to four groups by dimensions for the Timeseries, Table, Tree Map, and Pie Chart visualizations.
  - You can add more group by dimensions by clicking the + sign next to the by field. The first by value will be the primary grouping, the second by value will be the secondary grouping, and so on.

  - Q1: Build the following query and visualize the results as a table.

    Search for service:cart-checkout

    Show Sum of Cart Item Quantity and Sum of Cart Total by Customer ID limit to top 10 and Cart ID limit to top 10

    In this case, you've created a table of the quantity of items and total price of each cart that a customer checks out.

    <img width="1786" height="776" alt="image" src="https://github.com/user-attachments/assets/f93686e7-1cf3-4c83-b47e-62d70e0d1b2b" />

  - Q2: Now, build a table that lists the count unique of Cart ID and the sum of Cart Totals by Customer ID and Checkout Status.

    <img width="1746" height="807" alt="image" src="https://github.com/user-attachments/assets/8689a33b-6706-48db-93e2-e15d86ebaba0" />

## Alias a Facet
  - Sometimes your logs use different names for things that mean the same thing. For example, one log might use @cart.url and another might use @http.url_path, even though both are basically referring to a URL.
  - To make your searches easier, you can alias one attribute to another.
  
    Aliasing means: “Treat this attribute as if it were that other attribute.”

    So if you alias @cart.url → @http.url_path, then anytime you search using @http.url_path, Datadog will also include the values from @cart.url.

    - Q :Create a Table of the top 10 counts of all logs from service:(webserver OR cart-logger) grouped by URL Path, by Service, and by Status.
      <img width="1743" height="771" alt="image" src="https://github.com/user-attachments/assets/2c4c2101-51d7-474c-8ca7-e12402f6c7c3" />



    ### References
https://learn.datadoghq.com/courses/enhance-log-querying
