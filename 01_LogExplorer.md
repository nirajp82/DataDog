# Log Explorer

Log Explorer is the central interface for querying, filtering, and analyzing your ingested logs.
This documentation explains **all supported search syntax**, including tags, attributes, full-text search, Boolean logic, CIDR queries, wildcards, numerical filters, and more.

Visual diagrams and examples are included throughout.

---

# **TABLE OF CONTENTS**

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

# 📌 **Overview**

Log Explorer lets you search logs using:

```
tags         → key:value
attributes   → @key:value
full-text    → *:value
wildcards    → * and ?
phrases      → "multiple words"
Boolean      → AND, OR, -
CIDR         → CIDR(attribute, block)
```

Everything in this document builds on those core principles.

---

# **Search Term Basics**

### There are two types of search terms:

| Term Type   | Example         | Meaning                          |
| ----------- | --------------- | -------------------------------- |
| Single term | `hello`         | Matches “hello” anywhere allowed |
| Sequence    | `"hello world"` | Matches the exact phrase         |

### Tag & Attribute Formats

| Type      | Format       | Example             |
| --------- | ------------ | ------------------- |
| Tag       | `key:value`  | `service:frontend`  |
| Attribute | `@key:value` | `@http.method:POST` |

**Example query:**

```
service:webstore @http.method:POST
```

---

# **Wildcard Search**

Wildcards allow partial matches.

---

## ⭐ **Important Note (Simplified)**

Wildcards only work **outside** of quotes:

* `"*test*"` → searches for the **literal** text `*test*`
* `*test*` → matches any message containing **test**

---

### Multi-character wildcard: `*`

```
service:web*      → services beginning with "web"
web*              → messages starting with "web"
*web              → messages ending with "web"
service:*mongo    → services ending in "mongo"
*NETWORK*         → messages containing "NETWORK"
```

### Single-character wildcard: `?`

Matches one special character or space:

```
@my_attribute:hello?world
```

---

# **Boolean Operators**

Operators are **case-sensitive**.

| Operator | Meaning                        | Example                        |
| -------- | ------------------------------ | ------------------------------ |
| `AND`    | All terms must match (default) | `authentication AND failure`   |
| `OR`     | Either term may match          | `authentication OR password`   |
| `-`      | Exclude matches                | `authentication AND -password` |

### Examples

```
service:(webstore OR mobile-store) -status:info
availability-zone:(us-central1-f AND us-central1-c AND us-central1-a)
```

---

# **Full-text Search**

Use `*:term` to search **across all attributes**, including message.

### Examples

| Syntax            | Type      | Description                             |
| ----------------- | --------- | --------------------------------------- |
| `*:hello`         | Full-text | Search all attributes for “hello”       |
| `hello`           | Free text | Search message, title, and error fields |
| `*:hello*`        | Full-text | Begins with “hello”                     |
| `*:"hello world"` | Full-text | Exact phrase match                      |

⚠️ Not allowed in:
Index filters, archive filters, pipeline filters, rehydration, Live Tail.

---

# **Escaping Special Characters**

Escape the following using `\`:

```
- ! && || > >= < <= ( ) { } [ ] " * ? : \ # <space>
```

### Examples

```
@my_attribute:hello\:world
@my_attribute:"hello:world"
@my_attribute:hello?world
```

Notes:

* `/` is **not** a special character
* `@` cannot be searched (reserved for attributes)
* To search special characters in messages, parse them into attributes first

---

# **Attribute Search**

Attributes begin with `@`.

### Example

```
@url:www.datadoghq.com
```

### More Examples

```
@http.url_details.path:"/api/v1/test"
@http.url:/api\-v1/*
@http.status_code:[200 TO 299]
-@http.status_code:*
```

Notes:

* Attribute searches are case-sensitive
* Facets not required
* For case-insensitive search use full-text (`*:value`)

---

# **CIDR Queries**

Use CIDR to filter IP ranges.

### Examples

```
CIDR(@network.client.ip, 13.0.0.0/8)
CIDR(@network.ip.list, 13.0.0.0/8, 15.0.0.0/8)
source:pan.firewall evt.name:reject CIDR(@network.client.ip, 13.0.0.0/8)
source:vpc NOT(CIDR(@network.client.ip, 13.0.0.0/8)) CIDR(@network.destination.ip, 15.0.0.0/8)
```

Supports IPv4 and IPv6.

---

# **Numerical Attribute Search**

Must be added as a facet first.

### Examples

```
@http.response_time:>100
@http.status_code:[400 TO 499]
```

---

# **Tag Search**

### Standard tag examples

```
test
env:(prod OR test)
(env:prod AND -version:beta)
```

### Tags not using `key:value` format

```
tags:<MY_TAG>
```

---

# **Array Search**

Arrays are searchable.

### Simple array example

```
users.names:Peter
```

### Array of JSON objects (CloudWatch example)

```
@Event.EventData.Data.Name:ObjectServer
```

---

# **Calculated Fields**

Use `#` to reference a calculated field.

### Example

```
#latency:>500
```

These can be used in search, visualization, aggregation, or nested calculated fields.

---

# 📊 **Diagrams / Visual Aids**

---

## **1. Search Flow Overview**

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

---

## **2. Wildcard Behavior**

```
Outside quotes → wildcard active
Inside quotes  → wildcard ignored

Examples:
  "*test*" → literal
  *test*  → wildcard match
```

---

## **3. Boolean Logic Diagram**

```
A AND B      → intersection
A OR B       → union
A - B        → A excluding B
```

```
      A ∩ B               A ∪ B            A minus B
   ┌──────────┐       ┌──────────┐      ┌──────────┐
   │    ●     │       │●        ●│      │   ●       │
   └──────────┘       └──────────┘      └──────────┘
```

---

## **4. CIDR Matching Diagram**

```
CIDR(@ip, 10.0.0.0/8)

Checks:
 10.x.x.x → match
 192.x.x.x → no match
```

---

# 🧾 **CHEAT SHEET**

### **Basic Formats**

```
tag: value              → service:webstore
attribute: value        → @http.method:POST
phrase                  → "user login"
```

### **Wildcards**

```
*test*                  → contains "test"
hello?world             → matches "hello world"
```

### **Boolean**

```
A AND B
A OR B
A -B
```

### **Full-text**

```
*:error
*:login*
*:"hello world"
```

### **Attributes**

```
@url:/api/v1/*
@status_code:[200 TO 299]
-@debug:*
```

### **CIDR**

```
CIDR(@ip, 10.0.0.0/8)
```

### **Numerical**

```
@latency:>100
@status:[400 TO 499]
```

### **Tags**

```
env:prod
tags:<LEGACY_TAG>
```

### **Arrays**

```
users.names:Peter
@Event.EventData.Data.Name:ObjectServer
```

### **Calculated Fields**

```
#duration:>500
```
