# Log Explorer

Log Explorer is the central interface for querying, filtering, and analyzing your ingested logs.
This documentation explains **all supported search syntax**, including tags, attributes, full-text search, Boolean logic, CIDR queries, wildcards, numerical filters, arrays, calculated fields, and more.

Visual diagrams and examples are included throughout.

---

# Table of Contents

1. [Overview](#overview)
2. [Search Term Basics](#search-term-basics)
3. [Wildcard Search](#wildcard-search)
4. [Boolean Operators](#boolean-operators)
5. [Full-text Search](#full-text-search)
6. [Escaping Special Characters](#escaping-special-characters)
7. [Attribute Search](#attribute-search)
8. [CIDR Queries](#cidr-queries)
9. [Numerical Attribute Search](#numerical-attribute-search)
10. [Tag Search](#tag-search)
11. [Array Search](#array-search)
12. [Calculated Fields](#calculated-fields)
13. [Diagrams / Visual Aids](#diagrams--visual-aids)
14. [Cheat Sheet](#cheat-sheet)

---

# 📌 Overview

Log Explorer lets you search logs using:

| Type       | Syntax                   | Description                               |
| ---------- | ------------------------ | ----------------------------------------- |
| Tags       | `key:value`              | Search logs with a specific tag           |
| Attributes | `@key:value`             | Search logs with an attribute             |
| Full-text  | `*:value`                | Search across all fields and message text |
| Wildcards  | `*` and `?`              | Partial matches                           |
| Phrases    | `"multiple words"`       | Exact phrase search                       |
| Boolean    | `AND`, `OR`, `-`         | Combine or exclude terms                  |
| CIDR       | `CIDR(attribute, block)` | Filter IP ranges                          |

---

# Search Term Basics

| Term Type   | Example         | Meaning                                                                                            |
| ----------- | --------------- | -------------------------------------------------------------------------------------------------- |
| Single term | `hello`         | Matches "hello" anywhere in the log                                                                |
| Sequence    | `"hello world"` | Matches the **exact phrase**. Spaces and special characters **do not need escaping** inside quotes |

### Tag & Attribute Formats

| Type      | Format       | Example             |
| --------- | ------------ | ------------------- |
| Tag       | `key:value`  | `service:frontend`  |
| Attribute | `@key:value` | `@http.method:POST` |

**Example Query:**

```
service:webstore @http.method:POST
```

---

# Wildcard Search

Wildcards allow partial matches and work **only outside quotes**.

| Wildcard | Example       | Meaning                                          |
| -------- | ------------- | ------------------------------------------------ |
| `*`      | `*test*`      | Matches any text containing "test"               |
| `*`      | `web*`        | Matches text starting with "web"                 |
| `*`      | `*web`        | Matches text ending with "web"                   |
| `?`      | `hello?world` | Matches any **single character** in place of `?` |

> `"*test*"` → literal, does not expand wildcards

---

# Boolean Operators

Operators are **case-sensitive**.

| Operator | Meaning                        | Example                        | Explanation                                                           |
| -------- | ------------------------------ | ------------------------------ | --------------------------------------------------------------------- |
| `AND`    | All terms must match (default) | `authentication AND failure`   | Returns logs containing **both** "authentication" and "failure"       |
| `OR`     | Either term may match          | `authentication OR password`   | Returns logs containing **either** "authentication" or "password"     |
| `-`      | Exclude matches                | `authentication AND -password` | Returns logs containing "authentication" **but excluding** "password" |

### Example Queries

```
service:(webstore OR mobile-store) -status:info
availability-zone:(us-central1-f AND us-central1-c AND us-central1-a)
```

---

# Full-text Search

Use `*:term` to search **across all attributes**, including the message.

| Syntax            | Type      | Meaning                                 |
| ----------------- | --------- | --------------------------------------- |
| `*:hello`         | Full-text | Search all attributes for "hello"       |
| `hello`           | Free text | Search message, title, and error fields |
| `*:hello*`        | Full-text | Attributes starting with "hello"        |
| `*:"hello world"` | Full-text | Exact phrase search                     |

> Not allowed in: Index filters, archive filters, pipeline filters, rehydration, Live Tail

---

# Escaping Special Characters

Escape special characters using `\`:

```
- ! && || > >= < <= ( ) { } [ ] " * ? : \ # <space>
```

| Query                         | Meaning                                               |
| ----------------------------- | ----------------------------------------------------- |
| `@my_attribute:hello\:world`  | Matches literal "hello:world" — colon escaped         |
| `@my_attribute:"hello:world"` | Matches exact phrase "hello:world" — no escape needed |
| `@my_attribute:hello?world`   | `?` acts as single-character wildcard                 |

> Notes: `/` is **not** special, `@` cannot be searched directly

---

# Attribute Search

| Example                                 | Meaning                           |
| --------------------------------------- | --------------------------------- |
| `@url:www.datadoghq.com`                | Logs with this URL                |
| `@http.url_details.path:"/api/v1/test"` | Exact path match                  |
| `@http.url:/api\-v1/*`                  | URL starts with `/api-v1/`        |
| `@http.status_code:[200 TO 299]`        | Status 200–299                    |
| `-@http.status_code:*`                  | Exclude logs with any status code |

> Notes: Attribute searches are case-sensitive. For case-insensitive, use full-text search (`*:value`)

---

# CIDR Queries

| Query Example                                                                                    | Meaning                                           | Example IPs                       |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------- | --------------------------------- |
| `CIDR(@ip, 10.0.0.0/8)`                                                                          | IP in `10.0.0.0/8` range                          | 10.1.2.3, 10.255.255.1            |
| `CIDR(@network.client.ip, 13.0.0.0/8)`                                                           | Client IP in `13.0.0.0/8`                         | 13.0.1.5, 13.25.100.200           |
| `CIDR(@network.ip.list, 13.0.0.0/8, 15.0.0.0/8)`                                                 | Any IP in list in `13.x` or `15.x`                | 13.1.2.3, 15.0.0.10               |
| `source:pan.firewall evt.name:reject CIDR(@network.client.ip, 13.0.0.0/8)`                       | Firewall reject + client IP in `13.x`             | 13.10.20.30                       |
| `source:vpc NOT(CIDR(@network.client.ip, 13.0.0.0/8)) CIDR(@network.destination.ip, 15.0.0.0/8)` | Client IP **not** in 13.x, destination IP in 15.x | Client: 12.5.6.7 → Dest: 15.1.2.3 |

> Supports IPv4 and IPv6

---

# Numerical Attribute Search

| Example                | Meaning             |
| ---------------------- | ------------------- |
| `@latency:>100`        | Latency > 100       |
| `@status:[400 TO 499]` | Status code 400–499 |

> Must be added as a facet first

---

# Tag Search

| Example                        | Meaning                                    |
| ------------------------------ | ------------------------------------------ |
| `test`                         | Logs tagged "test"                         |
| `env:(prod OR test)`           | Environment = prod or test                 |
| `(env:prod AND -version:beta)` | Environment = prod, excluding beta version |
| `tags:<MY_TAG>`                | Legacy tag search                          |

---

# Array Search

| Example                                   | Meaning                               |
| ----------------------------------------- | ------------------------------------- |
| `users.names:Peter`                       | Array contains Peter                  |
| `@Event.EventData.Data.Name:ObjectServer` | JSON object field Name = ObjectServer |

---

# Calculated Fields

| Example         | Meaning                        |
| --------------- | ------------------------------ |
| `#latency:>500` | Calculated field latency > 500 |

> Can be used in search, visualization, aggregation, or nested calculated fields

---

# Diagrams / Visual Aids

## 1. Search Flow

```
┌──────────────────────────────┐
│        Search Query          │
└───────────────┬──────────────┘
                │
   ┌────────────┴───────────────┐
   │                            │
Tags (key:value)          Attributes (@key:value)
   │                            │
   └────────────┬───────────────┘
                │
     Boolean + Wildcards + Full-text
                │
                ▼
        Filtered Log Results
```

## 2. Wildcard Behavior

```
Outside quotes → wildcard active
Inside quotes  → wildcard ignored
```

## 3. Boolean Logic

```
A AND B → intersection
A OR B → union
A - B → difference
```

```
      A ∩ B               A ∪ B            A minus B
   ┌──────────┐       ┌──────────┐      ┌──────────┐
   │    ●     │       │●        ●│      │   ●       │
   └──────────┘       └──────────┘      └──────────┘
```

## 4. CIDR Matching

```
CIDR(@ip, 10.0.0.0/8)
Matches 10.x.x.x
Does not match 192.x.x.x
```

---

# Cheat Sheet

| Feature           | Example                                                                                          | Explanation                             |
| ----------------- | ------------------------------------------------------------------------------------------------ | --------------------------------------- |
| Tag               | `service:webstore`                                                                               | Logs with `service:webstore`            |
| Tag               | `env:prod`                                                                                       | Environment = prod                      |
| Tag               | `tags:<LEGACY_TAG>`                                                                              | Legacy tag search                       |
| Attribute         | `@http.method:POST`                                                                              | Logs with POST method                   |
| Attribute         | `@url:/api/v1/*`                                                                                 | URL starts with `/api/v1/`              |
| Attribute         | `@http.status_code:[200 TO 299]`                                                                 | Status 200–299                          |
| Attribute         | `-@debug:*`                                                                                      | Exclude debug logs                      |
| Phrase / Sequence | `"user login"`                                                                                   | Exact phrase search                     |
| Wildcard `*`      | `*test*`                                                                                         | Contains `test`                         |
| Wildcard `?`      | `hello?world`                                                                                    | Single-character wildcard               |
| Boolean AND       | `A AND B`                                                                                        | Logs containing both A and B            |
| Boolean OR        | `A OR B`                                                                                         | Logs containing either A or B           |
| Boolean NOT       | `A -B`                                                                                           | Logs containing A, excluding B          |
| Full-text         | `*:error`                                                                                        | Search all attributes for `error`       |
| Full-text         | `*:login*`                                                                                       | Attributes starting with `login`        |
| Full-text         | `*:"hello world"`                                                                                | Exact phrase across all attributes      |
| Escaping          | `@my_attribute:hello\:world`                                                                     | Escape colon                            |
| Escaping          | `@my_attribute:"hello:world"`                                                                    | Exact phrase, no escape needed          |
| Escaping          | `@my_attribute:hello?world`                                                                      | Single-character wildcard               |
| CIDR              | `CIDR(@ip, 10.0.0.0/8)`                                                                          | IP in 10.0.0.0/8 range                  |
| CIDR              | `CIDR(@network.client.ip, 13.0.0.0/8)`                                                           | Client IP in 13.0.0.0/8                 |
| CIDR              | `CIDR(@network.ip.list, 13.0.0.0/8, 15.0.0.0/8)`                                                 | Any IP in list in 13.x or 15.x          |
| CIDR              | `source:vpc NOT(CIDR(@network.client.ip, 13.0.0.0/8)) CIDR(@network.destination.ip, 15.0.0.0/8)` | Client not in 13.x, destination in 15.x |
| Numerical         | `@latency:>100`                                                                                  | Latency > 100                           |
| Numerical         | `@status:[400 TO 499]`                                                                           | Status code 400–499                     |
| Array             | `users.names:Peter`                                                                              | Array contains Peter                    |
| Array             | `@Event.EventData.Data.Name:ObjectServer`                                                        | JSON object field Name = ObjectServer   |
| Calculated        | `#duration:>500`                                                                                 | Calculated field > 500                  |
