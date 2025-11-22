# Log Explorer

Log Explorer is the central interface for querying, filtering, and analyzing your ingested logs.
This documentation explains **all supported search syntax**, including tags, attributes, full-text search, Boolean logic, CIDR queries, wildcards, numerical filters, arrays, calculated fields, and more.

---

# Table of Contents

1. [Overview](#overview)
2. [Fields Coverage (with Examples)](#fields-coverage-with-examples)
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

---

# 📌 Overview

Log Explorer lets you search logs using:

| Type           | Syntax                   | Description & Fields Searched                                                                                                                                                               |
| -------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Tags**       | `key:value`              | Search logs with a specific tag. Only the **tags** are searched. Case-sensitive.                                                                                                            |
| **Attributes** | `@key:value`             | Search logs with a specific attribute. Only **attributes** are searched. Case-sensitive.                                                                                                    |
| **Full-text**  | `*:value`                | Search **across all fields**, including **log messages, tags, and attributes**. Case-insensitive for log messages; tags/attributes are searched in their original case. Matches substrings. |
| **Wildcards**  | `*` and `?`              | Partial matches. `*` matches multiple characters, `?` matches a single character. Works in **log messages** and attribute values. Wildcards do **not** work inside quotes.                  |
| **Phrases**    | `"multiple words"`       | Exact phrase search. Searches **log messages** only. Spaces and special characters are treated literally inside quotes.                                                                     |
| **Boolean**    | `AND, OR, -`             | Combine or exclude terms. Applies to **log messages by default**. `AND` ensures both terms exist, `OR` allows either, `-` excludes terms.                                                   |
| **CIDR**       | `CIDR(attribute, block)` | Filter IP ranges. Only searches **attributes containing IPs**. Supports IPv4 and IPv6.                                                                                                      |

> **Case behavior**:
>
> * Log messages: **case-insensitive**
> * Tags / Attributes: **case-sensitive**
> * Full-text (`*:`): **case-insensitive** across log messages, tags, and attributes

---

# Fields Coverage

| Search Type            | Log Message Example          | Tag Example            | Attribute Example                 | Case Behavior & Explanation                                                                                                 |
| ---------------------- | ---------------------------- | ---------------------- | --------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **Tag**                | ❌                            | `service:webstore`     | ❌                                 | Case-sensitive. Only searches **tags**. Does **not** match log messages or attributes.                                      |
| **Attribute**          | ❌                            | ❌                      | `@http.method:POST`               | Case-sensitive. Only searches **attributes**. Does **not** match log messages or tags.                                      |
| **Full-text**          | `Error: failed login`        | `service:webstore`     | `@http.method:POST`               | Case-insensitive for log messages; matches **tags and attributes as well**. Substrings match (`helloWorld`, `Hello world`). |
| **Phrase / Sequence**  | `"login failed"`             | ❌                      | ❌                                 | Case-insensitive in **log messages only**. Use `*:"login failed"` for full-text across all fields.                          |
| **Boolean (AND/OR/-)** | `authentication AND failure` | `env:prod OR env:test` | `@http.status_code:200 -@debug:*` | Log messages are case-insensitive. Tags/attributes searched **only if explicitly mentioned**.                               |
| **Wildcard**           | `*error*`                    | `service:web*`         | `@http.url:/api/*`                | Works **outside quotes**. Log messages: case-insensitive. Tags/attributes: case-sensitive.                                  |

> ✅ Field is searched
> ❌ Field is not searched

---

# Search Term Basics

| Term Type   | Example         | Behavior & Case Sensitivity                                                                                                                                               |
| ----------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Single term | `hello`         | Matches anywhere in **log message**, case-insensitive. Substrings match: `helloWorld`, `Hello world`. Does **not** match in tags/attributes unless using full-text (`*:`) |
| Sequence    | `"hello world"` | Matches **exact phrase** in **log message**, case-insensitive. Does not match in tags/attributes unless using full-text (`*:"hello world"`)                               |

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

| Operator | Example                        | Behavior & Explanation                                                                                                                                                                                                                                                                                                                                                                |
| -------- | ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AND`    | `authentication AND failure`   | Logs **must contain both terms** to match. <br>**Fields affected:** <br>- **Log messages:** searched automatically, **case-insensitive**. <br>- **Tags/Attributes:** searched **only if explicitly mentioned**, and **case-sensitive**. <br>**Example:** `"User authentication failure"` matches log message, but not tag `Service:AuthenticationFailure` unless explicitly included. |
| `OR`     | `authentication OR password`   | Logs **must contain either term**. <br>**Fields affected:** same as above. <br>**Example:** matches `"authentication succeeded"` or `"password expired"`; will not match tags/attributes unless explicitly included.                                                                                                                                                                  |
| `-`      | `authentication AND -password` | Includes logs that contain `"authentication"` **but exclude** `"password"`. <br>**Fields affected:** log messages case-insensitive; tags/attributes only if explicitly searched. <br>**Example:** `"authentication failed for user"` included, `"authentication failed due to password policy"` excluded.                                                                             |

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

> Attribute searches are case-sensitive. For case-insensitive search, use full-text (`*:value`).

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

> Tags are case-sensitive.

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

