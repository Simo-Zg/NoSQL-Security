# 🛡️ MongoDB NoSQL Injection Lab — Séance 4

## 📌 Overview

This repository documents the practical work carried out during **Séance 4 — NoSQL Injection, Safe Query Construction & Minimal Projection** in a cybersecurity-oriented MongoDB lab.

The objective of this work was to **understand how unsafe application code can transform user input into dangerous MongoDB query logic**, then progressively secure the API by applying validation, explicit filter construction, operator blocking, allow-listed updates, and minimal projections.

The lab focuses on:

* NoSQL injection fundamentals in MongoDB
* Vulnerable Node.js / Express API patterns
* Authentication bypass with `$ne`
* Data exfiltration through `$regex` and `$gt`
* Mass assignment risks during profile updates
* Safe query construction with `safeFilter`
* Type validation and MongoDB operator filtering
* Least-data responses through projection
* Homework validation commands and expected results

The project includes:

* ✔️ Full LuaLaTeX report (PDF + source)
* ✔️ Séance 4 lab support file
* ✔️ Screenshots used as evidence
* ✔️ Minimal Node.js / Express applications for each part
* ✔️ MongoDB seed data for one-collection reproduction
* ✔️ Linux-style homework commands

---

## 🧠 Key Concepts Covered

### 🔹 NoSQL Injection Fundamentals

* Difference between SQL injection and NoSQL injection
* MongoDB filters as JSON objects
* Injection of query operators instead of SQL syntax breaking
* Why MongoDB executes the filter it receives without knowing user intent

### 🔹 Vulnerable API Patterns

* Passing `req.body` directly into `find()`
* Using user-controlled values inside `findOne()` without type validation
* Applying `$set: req.body` during update operations
* Returning full MongoDB documents without projection

### 🔹 Controlled Injection Demonstrations

* `$ne` authentication bypass
* `$regex` empty-pattern log exfiltration
* `$gt` broad-match exfiltration
* Mass assignment through unauthorized `role` update

### 🔹 Defensive Application Design

* Validate input types before query construction
* Build filters field by field
* Reject objects where strings are expected
* Reject keys starting with `$`
* Use allow-lists for updateable fields
* Return only required fields with projection

---

## 🗂️ Repository Structure

```bash
.
├── README.md
│
├── report/
│   ├── seance4_nosql_report.pdf
│   └── seance4_nosql_report.tex
│
├── lab/
│   └── seance4_nosql_injection_securite.html
│
├── src/
│   ├── package.json
│   ├── part1-vulnerable-normal.js
│   ├── part2-vulnerable-injections.js
│   ├── part3-safe-filter.js
│   └── part4-safe-projection.js
│
├── data/
│   └── seed_items.mongodb.js
│
├── commands/
│   └── homework_commands_linux.md
│
├── screenshots/
│   └── screenshots.zip
│
└── assets/
    └── fig_*.png
```

---

## ⚙️ Environment Setup

### 🐳 Run MongoDB on the lab port

The light version of this lab uses MongoDB on port `5017`.

```bash
docker run -d \
  --name mongo_nosqli_lab \
  -p 127.0.0.1:5017:27017 \
  mongo:7
```

### 🔌 Connect with mongosh

```bash
mongosh "mongodb://127.0.0.1:5017/nosqli_lab"
```

### 📦 Install Node.js dependencies

```bash
cd src
npm install
```

---

## 🧪 Database and Collection Model

The original Séance 4 support works with the SOC-lite idea and MongoDB collections such as `users` and `security_logs`.

For a lighter and faster reproduction, this GitHub version uses **one database and one collection**:

```txt
MongoDB URI: mongodb://127.0.0.1:5017/nosqli_lab
Database:    nosqli_lab
Collection:  items
```

The `items` collection contains two logical document types:

* `kind: "user"` → user accounts used for authentication tests
* `kind: "log"` → security logs used for search and exfiltration tests

---

## 📥 Seed the Collection

From the repository root:

```bash
mongosh "mongodb://127.0.0.1:5017/nosqli_lab" < data/seed_items.mongodb.js
```

Or inside `mongosh`:

```javascript
load("data/seed_items.mongodb.js")
```

Verify the inserted data:

```javascript
db.items.find({}, { _id: 0 }).pretty()
```

---

## 🚀 Run the Lab Applications

Each part is intentionally separated into a very small application so the vulnerable and secure behaviors are easy to compare.

### Part 1 — Vulnerable API with normal requests

```bash
cd src
node part1-vulnerable-normal.js
```

Normal log search:

```bash
curl -s -X POST http://localhost:3000/logs/search \
  -H "Content-Type: application/json" \
  -d '{"src_ip":"192.168.1.10"}' | python3 -m json.tool
```

Normal login:

```bash
curl -s -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"motdepasse123"}' | python3 -m json.tool
```

---

### Part 2 — Vulnerable API with injection demonstrations

```bash
cd src
node part2-vulnerable-injections.js
```

#### `$ne` authentication bypass

```bash
curl -s -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":{"$ne":""}}' | python3 -m json.tool
```

Expected vulnerable result:

```json
{
  "success": true,
  "username": "alice",
  "role": "analyst"
}
```

#### `$regex` log exfiltration

```bash
curl -s -X POST http://localhost:3000/logs/search \
  -H "Content-Type: application/json" \
  -d '{"src_ip":{"$regex":""}}' | python3 -m json.tool
```

#### `$gt` broad-match exfiltration

```bash
curl -s -X POST http://localhost:3000/logs/search \
  -H "Content-Type: application/json" \
  -d '{"action":{"$gt":""}}' | python3 -m json.tool
```

#### Mass assignment attempt

```bash
curl -s -X POST http://localhost:3000/users/carol/profile \
  -H "Content-Type: application/json" \
  -d '{"email":"carol2@soc.local","role":"admin"}' | python3 -m json.tool
```

Verify the effect:

```bash
mongosh "mongodb://127.0.0.1:5017/nosqli_lab"
```

```javascript
db.items.findOne(
  { kind: "user", username: "carol" },
  { _id: 0, username: 1, email: 1, role: 1 }
)
```

---

### Part 3 — Safe filter and allow-listed update

```bash
cd src
node part3-safe-filter.js
```

The same `$ne` injection should now fail:

```bash
curl -s -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":{"$ne":""}}' | python3 -m json.tool
```

Expected secure result:

```json
{
  "error": "Invalid credentials format"
}
```

The same `$regex` injection should also fail:

```bash
curl -s -X POST http://localhost:3000/logs/search \
  -H "Content-Type: application/json" \
  -d '{"src_ip":{"$regex":""}}' | python3 -m json.tool
```

Expected secure result:

```json
{
  "error": "Invalid src_ip"
}
```

Safe email update:

```bash
curl -s -X POST http://localhost:3000/users/update-email \
  -H "Content-Type: application/json" \
  -d '{"username":"carol","email":"carol.new@soc.local"}' | python3 -m json.tool
```

Mass assignment attempt against the safe endpoint:

```bash
curl -s -X POST http://localhost:3000/users/update-email \
  -H "Content-Type: application/json" \
  -d '{"username":"carol","email":"nouveau@test.com","role":"admin"}' | python3 -m json.tool
```

Expected behavior: the email may change, but the role must not become `admin`.

---

### Part 4 — Safe projection / least data

```bash
cd src
node part4-safe-projection.js
```

```bash
curl -s -X POST http://localhost:3000/logs/search \
  -H "Content-Type: application/json" \
  -d '{"src_ip":"192.168.1.10"}' | python3 -m json.tool
```

Expected exposed fields only:

```txt
ts
src_ip
action
status
```

The API response must not expose unnecessary fields such as:

```txt
_id
kind
severity
password_hash
internal_note
```

---

## 🧾 Homework Commands

The Linux-style homework commands are grouped in:

```bash
commands/homework_commands_linux.md
```

Covered tests:

* `$gt` injection on logs
* unexpected `role` field during login
* `hasNoMongoOperators()` validation logic
* projection with and without `.project()`
* secure `/users/update-email` endpoint

---

## ⚠️ Security Issues Addressed

### ❌ Direct use of `req.body` in `find()`

Risk:

* attacker controls the MongoDB filter
* injected operators such as `$regex` or `$gt` broaden the result set
* sensitive logs may be returned without authorization logic

✔️ Mitigation:

* construct filters explicitly field by field
* validate expected types
* reject objects where strings are expected

---

### ❌ Login bypass with `$ne`

Risk:

* password comparison is transformed into `password != ""`
* valid accounts can be returned without knowing the password
* authentication logic becomes controlled by attacker-supplied JSON

✔️ Mitigation:

* validate that username and password are strings
* never pass a password object into MongoDB
* in real systems, hash passwords and compare with a password hashing library

---

### ❌ Mass assignment

Risk:

* user changes fields that should never be client-controlled
* privilege escalation through `role: "admin"`
* silent update without application-level detection

✔️ Mitigation:

* create an allow-listed update object
* update only fields explicitly approved by the application
* never apply `$set: req.body` directly

---

### ❌ Overexposed API responses

Risk:

* internal fields leak through normal API responses
* documents expose `_id`, `password_hash`, internal SOC notes or metadata
* blocked injection is not enough if responses are still too verbose

✔️ Mitigation:

* use MongoDB projection
* return only the fields required by the client
* apply the least-data principle systematically

---

## 📊 Key Outcomes

* Built and tested a vulnerable Node.js / Express API
* Reproduced controlled NoSQL injection payloads locally
* Demonstrated authentication bypass through `$ne`
* Demonstrated broad log retrieval through `$regex` and `$gt`
* Demonstrated mass assignment risk with unauthorized `role` modification
* Implemented safe query construction with type validation
* Implemented allow-listed update logic
* Implemented projection to reduce exposed fields
* Produced a structured report with screenshots as evidence

---

## 🔐 Security Takeaways

* MongoDB hardening is necessary, but it does not replace application security
* NoSQL injection happens when user input becomes query logic
* JSON structure must be validated, not only accepted as valid syntax
* ORMs/ODMs such as Mongoose reduce risk but do not eliminate unsafe query construction
* Never trust `req.body`, `req.query`, or `req.params` as database filters
* Safe filters must be built by the server, not by the client
* Update operations must be based on allow-lists
* Projection is a security control, not only a performance optimization

---

## 📄 Report

The full technical report is available in the `report/` folder.

It includes:

* execution narrative following the Séance 2 report style
* explanation of the vulnerable and corrected flows
* screenshots used as proof of work
* homework command results and interpretations
* security checklist and lessons learned

Suggested main file:

📘 `report/seance4_nosql_report.pdf`

---

## 🖼️ Evidence Included

This repository is designed to document the work transparently.

Included evidence may contain:

* terminal outputs
* MongoDB command results
* curl requests and JSON responses
* vulnerable injection results
* safe-filter rejection results
* projection comparisons
* report pages and proof snapshots

---

## 🚀 Possible Improvements

* Replace plaintext passwords with bcrypt hashing
* Add JWT-based authentication flow
* Add automated tests for vulnerable vs safe endpoints
* Add a Docker Compose file for MongoDB and the API
* Add Mongoose versions of the same vulnerable and secure patterns
* Add centralized logging for rejected payloads
* Add rate limiting on authentication endpoints
* Extend the lab with RBAC-aware API authorization

---

## 👨‍💻 Author

**Mohammed ZGUIOUI**  
Cybersecurity / DevSecOps

---

## 📜 License

This project is shared for **educational and portfolio purposes**.

Before publishing publicly, make sure that:

* credentials are removed or rotated
* exported data is sanitized
* screenshots do not leak sensitive secrets
* database dumps contain only lab-safe content
* vulnerable endpoints are clearly marked as local demonstrations only
