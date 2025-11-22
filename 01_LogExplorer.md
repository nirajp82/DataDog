Got it! I’ve rewritten your full README, improving clarity on **fields searched, case sensitivity, full-text vs attribute search, and examples**, and made it logically structured. Here’s the final version ready for your use:

---

# Log Explorer

Log Explorer is the central interface for querying, filtering, and analyzing your ingested logs.
This documentation explains **all supported search syntax**, including tags, attributes, full-text search, Boolean logic, CIDR queries, wildcards, numerical filters, arrays, calculated fields, and more.

---

# Table of Contents

1. [Overview](#overview)
2. [Quick Examples](#quick-examples)
3. [Search Term Basics](#search-term-basics)
4. [Wildcard Search](#wildcard-search)
5. [Boolean Operators](#boolean-operators)
6. [Full-text Search](#full-text-search)
7. [Escaping Special Characters](#escaping-special-characters)
8. [Attribute Search](#attribute-search)
9. [CIDR Queries](#cidr-queries)
10. [Numerical Attribute Search](#numerical-attribute-search)
11. [Tag Search](#tag-search)
12. [Array Search](#array-search)
13. [Calculated Fields](#calculated-fields)
14. [Cheat Sheet](#cheat-sheet)

---

# 📌 Overview

Log Explorer lets you search logs using:

| Type           | Syntax                   | Description & Fields Searched                                                                                                                                              |
| -------------- | ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Tags**       | `key:value`              | Search logs with a specific tag. Only the **tags** are searched. Case-sensitive.                                                                                           |
| **Attributes** | `@key:value`             | Search logs with a specific attribute. Only **attributes** are searched. Case-sensitive.                                                                                   |
| **Full-text**  | `*:value`                | Search **across all fields**, including **log messages, tags, and attributes**. Case-insensitive for log messages, case-sensitive for tags/attributes. Matches substrings. |
| **Wildcards**  | `*` and `?`              | Partial matches. `*` matches multiple characters, `?` matches a single character. Works in **log messages** and attribute values. Wildcards do **not** work inside quotes. |
| **Phrases**    | `"multiple words"`       | Exact phrase search. Searches **log messages** only. Spaces and special characters are treated literally inside quotes.                                                    |
| **Boolean**    | `AND, OR, -`             | Combine or exclude terms. Applies to **log messages by default**. `AND` ensures both terms exist, `OR` allows either, `-` excludes terms.                                  |
| **CIDR**       | `CIDR(attribute, block)` | Filter IP ranges. Only searches **attributes containing IPs**. Supports IPv4 and IPv6.                                                                                     |

> **Case behavior**:
>
> * Log messages: **case-insensitive**
> * Tags / Attributes: **case-sensitive**
> * Full-text (`*:`): **case-insensitive** across messages, tags, and attributes

---

# Quick Examples

| Query                   | Matches in                 | Example Logs / Tags / Attributes                                                 | Notes                                                                                                 |
| ----------------------- | -------------------------- | -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `hello`                 | Log message only           | `"Hello World"`, `"say hello"`                                                   | Case-insensitive; **does not match tags/attributes** unless using full-text (`*:`)                    |
| `service:webstore`      | Tag only                   | Tag `service:webstore`                                                           | Case-sensitive; log messages ignored                                                                  |
| `@http.method:POST`     | Attribute only             | Attribute `@http.method:POST`                                                    | Case-sensitive; log messages ignored                                                                  |
| `*:hello`               | All fields                 | Log message: `"Hello world"`<br>Tag: `greeting:hello`<br>Attribute: `@msg:hello` | Case-insensitive in log messages, tags, attributes                                                    |
| `"user login"`          | Log message only           | `"User login successful"`                                                        | Exact phrase match; quotes required; does not match tags/attributes unless `*:` used                  |
| `A AND B`               | Log message (by default)   | Log containing `"A"` and `"B"`                                                   | Boolean AND; case-insensitive in log messages; tags/attributes only if explicitly searched            |
| `A OR B`                | Log message (by default)   | Log containing `"A"` or `"B"`                                                    | Boolean OR                                                                                            |
| `A -B`                  | Log message (by default)   | Log contains `"A"` but excludes `"B"`                                            | Boolean NOT                                                                                           |
| `*test*`                | Log message and attributes | `"unit test passed"`, attribute `@name:mytest`                                   | Wildcard `*`; matches multiple characters; case-insensitive in messages, case-sensitive in attributes |
| `@url:/api/v1/*`        | Attribute only             | Attribute `@url:/api/v1/users`                                                   | Wildcards work in attributes; case-sensitive                                                          |
| `CIDR(@ip, 10.0.0.0/8)` | Attribute only             | Attribute `@ip:10.12.3.4`                                                        | Matches IPv4 or IPv6 in the given range                                                               |

> **Tip:**
>
> * Use `*:` for **full-text search** across all fields.
> * Boolean operators (`AND`, `OR`, `-`) apply to **log messages by default**, not tags/attributes.
> * Attribute and tag searches are **case-sensitive** unless using `*:`.

---

# Search Term Basics

| Term Type   | Example         | Behavior & Case Sensitivity                                                                                                                                               |
| ----------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Single term | `hello`         | Matches anywhere in **log message**, case-insensitive. Substrings match: `helloWorld`, `Hello world`. Does **not** match in tags/attributes unless using full-text (`*:`) |
| Sequence    | `"hello world"` | Matches **exact phrase** in **log message**, case-insensitive. Does not match in tags/attributes unless using full-text (`*:"hello world"`)                               |

### Tag & Attribute Formats

| Type      | Format       | Example             | Matches in      | Case Sensitivity |
| --------- | ------------ | ------------------- | --------------- | ---------------- |
| Tag       | `key:value`  | `service:frontend`  | Tags only       | Case-sensitive   |
| Attribute | `@key:value` | `@http.method:POST` | Attributes only | Case-sensitive   |

**Example Query:**

```
service:webstore @http.method:POST
```

---

# Wildcard Search

Wildcards work **only outside quotes**.

| Wildcard   | Example       | Behavior & Case Sensitivity                                                                         |
| ---------- | ------------- | --------------------------------------------------------------------------------------------------- |
| `*`        | `*test*`      | Matches any sequence of characters. Log messages: case-insensitive; tags/attributes: case-sensitive |
| `*`        | `web*`        | Matches text starting with "web"                                                                    |
| `*`        | `*web`        | Matches text ending with "web"                                                                      |
| `?`        | `hello?world` | Matches exactly one character                                                                       |
| `"*test*"` | literal       | Wildcard ignored inside quotes                                                                      |

---

# Boolean Operators

| Operator | Example                        | Behavior & Explanation                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| -------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AND`    | `authentication AND failure`   | Logs **must contain both terms** to match. <br>**Fields affected:** <br>- **Log messages:** searched automatically, **case-insensitive**. <br>- **Tags/Attributes:** searched **only if explicitly mentioned**, and **case-sensitive**. <br>**Example:** `authentication AND failure` will match a log message containing `"User authentication failure"` but **will not** match a tag `Service:AuthenticationFailure` unless you explicitly include it. |
| `OR`     | `authentication OR password`   | Logs **must contain either term**. <br>**Fields affected:** same as above. <br>**Example:** matches a log message `"authentication succeeded"` or `"password expired"`; will not match tags/attributes unless explicitly included.                                                                                                                                                                                                                       |
| `-`      | `authentication AND -password` | Includes logs that contain `"authentication"` **but exclude** `"password"`. <br>**Fields affected:** <br>- Log messages: case-insensitive search. <br>- Tags/Attributes: only if explicitly searched. <br>**Example:** a log message `"authentication failed for user"` is included, but `"authentication failed due to password policy"` is excluded.                                                                                                   |

**Example Queries:**

```
service:(webstore OR mobile-store) -status:info
availability-zone:(us-central1-f AND us-central1-c AND us-central1-a)
```

---

# Full-text Search

Use `*:term` to search **everything** (messages, tags, attributes). Always **case-insensitive**.

| Syntax            | Example                                      | Notes            |
| ----------------- | -------------------------------------------- | ---------------- |
| `*:hello`         | ✅ Matches anywhere in logs, tags, attributes | Case-insensitive |
| `*:hello*`        | ✅ Matches attributes starting with hello     | Case-insensitive |
| `*:"hello world"` | ✅ Exact phrase anywhere                      | Case-insensitive |

> Not allowed in index filters, archive filters, pipeline filters, rehydration, Live Tail

---

# Escaping Special Characters

Escape using `\`:

```
- ! && || > >= < <= ( ) { } [ ] " * ? : \ # <space>
```

| Query                         | Meaning                                               |
| ----------------------------- | ----------------------------------------------------- |
| `@my_attribute:hello\:world`  | Matches literal `hello:world` — colon escaped         |
| `@my_attribute:"hello:world"` | Matches exact phrase `hello:world` — no escape needed |
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

> Attribute searches are **case-sensitive**. For case-insensitive search, use **full-text (`*:value`)**.
> Attribute search only searches the specified attribute. Log message and other fields are ignored.

---

# CIDR Queries

| Query Example                                                                                    | Meaning                                 | Example IPs                       |
| ------------------------------------------------------------------------------------------------ | --------------------------------------- | --------------------------------- |
| `CIDR(@ip, 10.0.0.0/8)`                                                                          | IP in `10.0.0.0/8` range                | 10.1.2.3, 10.255.255.1            |
| `CIDR(@network.client.ip, 13.0.0.0/8)`                                                           | Client IP in `13.0.0.0/8`               | 13.0.1.5, 13.25.100.200           |
| `CIDR(@network.ip.list, 13.0.0.0/8, 15.0.0.0/8)`                                                 | Any IP in list in `13.x` or `15.x`      | 13.1.2.3, 15.0.0.10               |
| `source:pan.firewall evt.name:reject CIDR(@network.client.ip, 13.0.0.0/8)`                       | Firewall reject + client IP in `13.x`   | 13.10.20.30                       |
| `source:vpc NOT(CIDR(@network.client.ip, 13.0.0.0/8)) CIDR(@network.destination.ip, 15.0.0.0/8)` | Client not in 13.x, destination in 15.x | Client: 12.5.6.7 → Dest: 15.1.2.3 |

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

> Tags are **case-sensitive**. Tag search only searches the **tag fields**.

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


# Cheat Sheet (Final)

| Feature           | Example                                                                                          | Explanation / Notes                                                           |
| ----------------- | ------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| Tag               | `service:webstore`                                                                               | Logs with `service:webstore`, case-sensitive                                  |
| Tag               | `env:prod`                                                                                       | Environment = prod                                                            |
| Tag               | `tags:<LEGACY_TAG>`                                                                              | Legacy tag search                                                             |
| Attribute         | `@http.method:POST`                                                                              | Logs with POST method, case-sensitive                                         |
| Attribute         | `@url:/api/v1/*`                                                                                 | URL starts with `/api/v1/`, case-sensitive                                    |
| Attribute         | `@http.status_code:[200 TO 299]`                                                                 | Status 200–299                                                                |
| Attribute         | `-@debug:*`                                                                                      | Exclude debug logs                                                            |
| Phrase / Sequence | `"user login"`                                                                                   | Exact phrase search in log messages; use full-text (`*:`) for tags/attributes |
| Wildcard `*`      | `*test*`                                                                                         | Contains `test`                                                               |
| Wildcard `?`      | `hello?world`                                                                                    | Single-character wildcard                                                     |
| Boolean AND       | `A AND B`                                                                                        | Logs containing both A and B                                                  |
| Boolean OR        | `A OR B`                                                                                         | Logs containing either A or B                                                 |
| Boolean NOT       | `A -B`                                                                                           | Logs containing A, excluding B                                                |
| Full-text         | `*:error`                                                                                        | Search all attributes, tags, and log messages, case-insensitive               |
| Full-text         | `*:login*`                                                                                       | Attributes/log messages starting with `login`                                 |
| Full-text         | `*:"hello world"`                                                                                | Exact phrase across all attributes and log messages                           |
| Escaping          | `@my_attribute:hello\:world`                                                                     | Escape colon                                                                  |
| Escaping          | `@my_attribute:"hello:world"`                                                                    | Exact phrase, no escape needed                                                |
| Escaping          | `@my_attribute:hello?world`                                                                      | Single-character wildcard                                                     |
| CIDR              | `CIDR(@ip, 10.0.0.0/8)`                                                                          | IP in 10.0.0.0/8 range                                                        |
| CIDR              | `CIDR(@network.client.ip, 13.0.0.0/8)`                                                           | Client IP in 13.0.0.0/8                                                       |
| CIDR              | `CIDR(@network.ip.list, 13.0.0.0/8, 15.0.0.0/8)`                                                 | Any IP in list in 13.x or 15.x                                                |
| CIDR              | `source:vpc NOT(CIDR(@network.client.ip, 13.0.0.0/8)) CIDR(@network.destination.ip, 15.0.0.0/8)` | Client not in 13.x, destination in 15.x                                       |
| Numerical         | `@latency:>100`                                                                                  | Latency > 100                                                                 |
| Numerical         | `@status:[400 TO 499]`                                                                           | Status code 400–499                                                           |
| Array             | `users.names:Peter`                                                                              | Array contains Peter                                                          |
| Array             | `@Event.EventData.Data.Name:ObjectServer`                                                        | JSON object field Name = ObjectServer                                         |
| Calculated        | `#duration:>500`                                                                                 | Calculated field > 500                                                        |

---

This version is **fully clear** on:

* Which **fields are searched** (log messages, tags, attributes)
* Case sensitivity rules
* Full-text search behavior
* Boolean operator behavior

