# 🛡️ NoSQL Security Lab — MongoDB SOC-Lite Platform

## 📌 Overview

This repository documents the practical work carried out during **Séance 2 — Modélisation NoSQL, Requêtes avancées & Index** within a cybersecurity-oriented MongoDB lab.

The objective was to design, implement, and secure a **SOC-lite (Security Operations Center) data platform** using MongoDB, focusing on:

* Data modeling (Embed vs Reference)
* Advanced querying and filtering
* Incident correlation with logs
* Index optimization
* Security-aware data design

The project includes:

* ✔️ Full LaTeX report (PDF + source)
* ✔️ MongoDB command execution logs
* ✔️ Exported collections (JSON)
* ✔️ Database archive (dump)
* ✔️ Screenshots of execution and results

---

## 🧠 Key Concepts Covered

### 🔹 NoSQL Data Modeling

* Embed vs Reference strategy
* One-to-Few / One-to-Many relationships
* Time-series data handling (logs)
* Controlled denormalization

### 🔹 SOC Data Architecture

* `users` → analysts and roles
* `security_logs` → raw events (append-only)
* `incidents` → correlated alerts with references to logs

### 🔹 Advanced MongoDB Operations

* Complex filtering (`$or`, `$in`, date ranges)
* Projection optimization
* Sorting and pagination
* Aggregation pipelines
* Update operators (`$push`, `$addToSet`)

### 🔹 Security Considerations

* Least privilege at data level
* Sensitive data exposure via embedding
* Separation of concerns (logs vs incidents)
* Query minimization (projection)

---

## 🗂️ Repository Structure

```bash
.
├── report/
│   ├── mongodb_lab2_report.pdf
│   └── mongodb_lab2_report.tex
│
├── data/
│   ├── ids_alerts.json
│   ├── incidents.json
│   ├── security_logs.json
│   └── users.json
│
├── archive/
│   └── db_backup.archive
│
├── commands/
│   └── Mongo-commands.txt
│
├── screenshots/
│   └── *.png
│
├── Lab2.html
│
└── README.md
```

---

## ⚙️ Environment Setup

### 🐳 Run MongoDB (Docker)

```bash
docker run -d \
  --name mongo-soc \
  -p 27017:27017 \
  mongo:7
```

### 🔌 Connect to MongoDB

```bash
docker exec -it mongo-soc mongosh
```

```javascript
use soc_lite
```

---

## 📥 Import Data (JSON Collections)

```bash
mongoimport --db soc_lite --collection users --file data/users.json --jsonArray
mongoimport --db soc_lite --collection security_logs --file data/security_logs.json --jsonArray
mongoimport --db soc_lite --collection incidents --file data/incidents.json --jsonArray
```

---

## 💾 Restore Full Database (Dump)

```bash
mongorestore dump/
```

---

## 🔍 Example Queries

### Get critical security events

```javascript
db.security_logs.find({ severity: "critical" })
```

### Filter logs within time range

```javascript
db.security_logs.find({
  timestamp: {
    $gte: new Date("2025-03-03T10:00:00Z"),
    $lte: new Date("2025-03-03T12:00:00Z")
  }
})
```

### Projection (minimal exposure)

```javascript
db.security_logs.find(
  { event_type: "login_failed" },
  { src_ip: 1, user_attempted: 1, timestamp: 1, _id: 0 }
)
```

### Latest logs (pagination pattern)

```javascript
db.security_logs.find().sort({ timestamp: -1 }).limit(3)
```

---

## ⚠️ Challenges & Fixes

### ❌ Error: Circular BSON Structure

```
BSONError: Cannot convert circular structure to BSON
```

✔️ **Fix**:
Use `.toArray()` before `.map()`:

```javascript
let logIds = db.security_logs
  .find({ src_ip: "192.168.1.105" })
  .toArray()
  .map(d => d._id);
```

---

### ❌ Error: Maximum Call Stack Size Exceeded

Cause: improper cursor handling

✔️ **Fix**:
Convert cursor → array before manipulation

---

## 📊 Key Results

* Successfully modeled a **scalable SOC database**
* Linked incidents ↔ logs using ObjectId references
* Implemented **secure query patterns**
* Demonstrated **data lifecycle (insert → update → query → correlate)**
* Validated model with real-world cybersecurity scenarios

---

## 🔐 Security Insights

* Avoid embedding large or sensitive datasets (logs)
* Always use **projection** to limit data exposure
* Separate collections improves **access control**
* Design impacts **attack surface**

---

## 📄 Report

The full technical report is available here:

📘 `report/mongodb_seance2_report.pdf`

It includes:

* Architecture diagrams
* Full explanations of modeling decisions
* Query breakdowns
* Security analysis
* Lab conclusions

---

## 🚀 Future Improvements

* Add **MongoDB indexes benchmarking (explain())**
* Integrate with **SIEM tools**
* Implement **role-based access control (RBAC)**
* Add **aggregation dashboards**
* Simulate real attack scenarios (DFIR use cases)

---

## 👨‍💻 Author

**Mohammed ZGUIOUI**
Cybersecurity & DevSecOps Enthusiast

---

## 📜 License

This project is for educational purposes only.
