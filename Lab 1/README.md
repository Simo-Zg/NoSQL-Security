# 🛡️ NoSQL Security Lab — Séance 1: Découvrir NoSQL & MongoDB

## 📌 Overview

This repository documents the practical work completed during **Séance 1 — Découvrir NoSQL & MongoDB** in a cybersecurity-oriented MongoDB lab.

The session introduced the foundations of **NoSQL data management** through a hands-on lab using **Docker**, **MongoDB**, and **mongosh**, with a focus on security-relevant use cases.

The main objective was to build and manipulate a small **SOC-style event dataset** while learning how MongoDB works in practice:

* SQL vs NoSQL fundamentals
* JSON document structure
* MongoDB architecture and Docker deployment
* CRUD operations on security events
* Advanced filtering, sorting, and projection
* Basic MongoDB attack surface awareness

The project includes:

* ✔️ Full LaTeX report (PDF + source)
* ✔️ MongoDB command execution logs
* ✔️ HTML course/lab support
* ✔️ Screenshots of execution and results
* ✔️ Practical answers for the final homework section

---

## 🧠 Key Concepts Covered

### 🔹 NoSQL Fundamentals

* Differences between SQL and NoSQL
* Document-oriented databases
* Flexible schema design
* JSON as the native data format

### 🔹 MongoDB Basics

* Database and collection creation
* Document insertion and retrieval
* CRUD lifecycle in `security_logs`
* Working with `ObjectId`, `Date`, and projections

### 🔹 Cybersecurity-Oriented Data Handling

* Security log storage
* Event severity classification
* Filtering suspicious events
* Minimal exposure through projection

### 🔹 Security Awareness

* MongoDB default exposure risks
* Network service visibility on port `27017`
* Difference between application access and native driver access
* Importance of secure-by-default deployment practices

---

## 🗂️ Repository Structure

```bash
.
├── report/
│   ├── mongodb_lab_report.pdf
│   └── mongodb_lab_report.tex
│
├── data/
│   └── security_logs.json
│
├── archive/
│   └── db_backup.archive
│
├── screenshots/
│   └── *.png
│
├── Lab1.html
│
└── README.md
```

---

## ⚙️ Environment Setup

### 🐳 Run MongoDB with Docker

```bash
docker run -d \
  --name mongo-noseclab \
  -p 27017:27017 \
  mongo:7
```

### 🔌 Connect to MongoDB shell

```bash
docker exec -it mongo-noseclab mongosh
```

```javascript
use noseclab
```

---

## 📥 Dataset Initialization

The lab uses a collection named `security_logs` to simulate basic security monitoring events.

Example insertion pattern:

```javascript
db.security_logs.insertMany([
  { src_ip: "10.10.10.10", action: "SSH Connection", severity: "high", timestamp: new Date() },
  { device: "server-db02", user: "admin", severity: "critical", action: "malware", timestamp: new Date() },
  { user: "alice", severity: "info", action: "logout", timestamp: new Date() }
])
```

---

## 🔍 Example Queries

### Get high and critical events after a given timestamp

```javascript
db.security_logs.find(
  {
    severity: { $in: ["high", "critical"] },
    timestamp: { $gte: ISODate("2026-03-04T10:00:00.000Z") }
  },
  {
    timestamp: 1,
    src_ip: 1,
    action: 1,
    severity: 1,
    _id: 0
  }
).sort({ timestamp: -1 })
```

### Mark logs from a suspicious IP for investigation

```javascript
db.security_logs.updateMany(
  { src_ip: "10.0.0.50" },
  { $set: { investigate: true, analyst: "Mohammed_Z" } }
)
```

### Verify updated logs

```javascript
db.security_logs.find({ src_ip: "10.0.0.50" })
```

### Delete a specific logout event

```javascript
db.security_logs.deleteOne({ user: "alice", action: "logout" })
```

### Count documents before and after deletion

```javascript
db.security_logs.countDocuments()
```

---

## 🌐 Surface d’attaque Verification

The final part of the session explored MongoDB exposure at the service level.

### Check whether MongoDB listens on port 27017

```bash
nmap -p 27017 localhost
```

### Test HTTP access against the native MongoDB port

```bash
curl -s http://localhost:27017
```

Expected result:

```text
It looks like you are trying to access MongoDB over HTTP on the native driver port.
```

### List databases from inside the container

```bash
docker exec -it mongo-noseclab mongosh --eval "db.adminCommand({ listDatabases: 1 })"
```

### Check port mapping from Docker

```bash
docker port mongo-noseclab
```

---

## ⚠️ Challenges & Observations

### ❌ Missing fields in projected results

Some returned documents do not display the same fields.

✔️ **Explanation**:
MongoDB only shows fields that actually exist in each document. If a document has no `src_ip`, it will not appear in the output even if included in the projection.

### ❌ `matchedCount` vs `modifiedCount`

During updates, both counters may differ in other scenarios.

✔️ **Explanation**:

* `matchedCount` = number of documents matched by the filter
* `modifiedCount` = number of documents actually changed

If values were already identical to the update payload, `modifiedCount` could be lower.

### ⚠️ Accessing MongoDB with `curl`

Using HTTP on port `27017` does not provide normal data access.

✔️ **Explanation**:
MongoDB does not expose a classic HTTP API on its native database port. The service speaks the MongoDB wire protocol, not HTTP.

---

## 📊 Key Results

* Successfully deployed MongoDB in Docker
* Created and populated the `security_logs` collection
* Executed CRUD operations on security-oriented data
* Practiced filtering, projection, sorting, and date-based querying
* Validated MongoDB service exposure on port `27017`
* Confirmed Docker port publication and local accessibility

---

## 🔐 Security Insights

* Exposing port `27017` unnecessarily increases attack surface
* A service being reachable does not mean it is safely configured
* Projection reduces unnecessary data disclosure
* Security logs should be queryable with minimal data exposure
* Even beginner labs benefit from explicit security validation steps

---

## 📄 Report

The full technical report is available in the `report/` directory.

It includes:

* Detailed explanation of the work completed
* Commands and observed outputs
* Screenshots as execution proof
* Final answers to the homework/security questions
* Technical interpretation of the MongoDB lab results

---

## 🚀 Future Improvements

* Add authentication and authorization to MongoDB
* Introduce indexes and compare performance with `explain()`
* Expand the dataset with incident correlation collections
* Add aggregation pipelines for SOC-style dashboards
* Simulate more realistic DFIR and detection workflows

---

## 👨‍💻 Author

**Mohammed ZGUIOUI**
Cybersecurity & DevSecOps Enthusiast

---

## 📜 License

This project is for educational purposes only.
