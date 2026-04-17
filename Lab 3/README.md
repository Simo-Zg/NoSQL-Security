<!-- File export mirrored to sandbox artifact in chat response -->

# 🛡️ MongoDB Hardening Lab — Séance 3

## 📌 Overview

This repository documents the practical work carried out during **Séance 3 — Hardening MongoDB: Authentication, RBAC, Network Exposure & Configuration Hygiene** in a cybersecurity-oriented MongoDB lab.

The objective of this work was to **secure a MongoDB deployment used for a SOC-lite platform** by applying database hardening measures and validating them with practical tests and visual proof.

The lab focuses on:

* MongoDB authentication and admin bootstrap
* Role-Based Access Control (RBAC)
* Least-privilege account design
* Network exposure reduction with Docker port binding
* Secret handling and configuration hygiene
* Verification of authorized vs refused access

The project includes:

* ✔️ Full LuaLaTeX report (PDF + source)
* ✔️ MongoDB exported collections (JSON)
* ✔️ Database archive / dump
* ✔️ Screenshots of executed commands and observed results
* ✔️ Lab support file and reproducible command examples

---

## 🧠 Key Concepts Covered

### 🔹 MongoDB Hardening Fundamentals

* Difference between database security and application security
* MongoDB attack surface and insecure default exposure
* Defense in depth for database services
* Secure-by-design deployment habits

### 🔹 Authentication & Access Control

* Root account initialization
* `authenticationDatabase` handling
* SCRAM-based authentication workflow
* Verification of denied anonymous access

### 🔹 RBAC & Least Privilege

* Separation of admin and operational accounts
* Read-only vs read-write roles
* Principle of least privilege
* Limits of built-in roles and need for custom roles

### 🔹 Secure Deployment Practices

* Docker-based MongoDB deployment
* Port binding to `127.0.0.1`
* Reduced network exposure
* Safer handling of credentials and environment variables

---

## 🗂️ Repository Structure

```bash
.
├── report/
│   ├── rapport_seance3_mongodb_style_seance2.pdf
│   └── rapport_seance3_mongodb_style_seance2.tex
│
├── data/
│   ├── users.json
│   ├── security_logs.json
│   └── incidents.json                # optional homework dataset export
│
├── archive/
│   └── db_backup.archive           # full MongoDB archive / backup
│
├── screenshots/
│   └── *.png
│
├── lab/
│   └── seance3_nosql_security_Partie3.html
│
└── README.md
```

---

## ⚙️ Environment Setup

### 🐳 Run MongoDB (secure local binding)

```bash
docker run -d \
  --name mongo_secure \
  -p 127.0.0.1:27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=adminSoc \
  -e MONGO_INITDB_ROOT_PASSWORD='Passw0rd_S3cure!' \
  mongo:7
```

### 🔌 Connect as administrator

```bash
mongosh "mongodb://adminSoc:Passw0rd_S3cure!@localhost:27017/admin"
```

### ❌ Anonymous access test

```bash
mongosh "mongodb://localhost:27017"
```

Then test a protected operation:

```javascript
show dbs
// expected: requires authentication
```

---

## 👥 Example Accounts Used in the Lab

### Administrator

```javascript
adminSoc
```

Used to:

* initialize and manage the instance
* create users and roles
* validate hardening configuration

### Operational / SOC Accounts

Examples discussed or created during the hardening workflow:

* `soc_app` → application-oriented account
* `soc_analyst_ro` → read-only analyst account
* role-scoped access to `soc_lite`

---

## 📥 Import Data (JSON Collections)

```bash
mongoimport --db soc_lite --collection users --file data/users.json --jsonArray
mongoimport --db soc_lite --collection security_logs --file data/security_logs.json --jsonArray
mongoimport --db soc_lite --collection incidents --file data/incidents.json --jsonArray
```

If the homework dataset is included:

```bash
mongoimport --db soc_lite --collection ids_alerts --file data/ids_alerts.json --jsonArray
```

---

## 💾 Restore Full Database Archive

```bash
mongorestore --archive=archive/soc_lite_dump.archive
```

If the dump is stored as a directory instead of a single archive:

```bash
mongorestore archive/
```

---

## 🔍 Example Validation Commands

### Verify that anonymous access is blocked

```javascript
show dbs
// MongoServerError: command listDatabases requires authentication
```

### Verify authenticated admin access

```javascript
db.adminCommand({ listDatabases: 1 })
```

### Create a scoped application user

```javascript
use soc_lite
db.createUser({
  user: "soc_app",
  pwd: "AppP@ss2024!",
  roles: [
    { role: "readWrite", db: "soc_lite" }
  ]
})
```

### Test correct authentication database

```bash
mongosh "mongodb://adminSoc:Passw0rd_S3cure!@localhost:27017/admin"
```

### Test incorrect authentication database

```bash
mongosh "mongodb://adminSoc:Passw0rd_S3cure!@localhost:27017/soc_lite"
# expected: Authentication failed
```

---

## ⚠️ Security Issues Addressed

### ❌ Insecure default exposure

Risk:

* MongoDB reachable without authentication
* Full read/write access if exposed
* Increased attack surface through broad port publishing

✔️ Mitigation:

* activate authentication
* create an admin account properly
* publish the port only on `127.0.0.1`

---

### ❌ Over-privileged database accounts

Risk:

* using one powerful account everywhere
* compromise of a single credential leads to broad impact

✔️ Mitigation:

* separate admin and operational users
* apply least privilege
* use RBAC to scope access by role

---

### ❌ Misuse of built-in roles

Risk:

* built-in roles such as `readWrite` can grant access to more collections than intended

✔️ Mitigation:

* evaluate built-in roles carefully
* use custom roles when collection-level granularity is required

---

## 📊 Key Outcomes

* Successfully secured a MongoDB lab instance with authentication enabled
* Verified the difference between anonymous and authenticated access
* Demonstrated the practical impact of local-only port binding
* Applied RBAC concepts to MongoDB user design
* Produced a structured hardening report with screenshots as evidence
* Prepared reusable database exports and archive files for publication and replay

---

## 🔐 Security Takeaways

* Database security is not the same as application security
* Authentication alone is not enough without least privilege
* Network exposure must be reduced as early as deployment time
* Hardening must be verified with positive and negative tests
* Built-in roles are convenient, but not always precise enough

---

## 📄 Report

The full technical report is available in the `report/` folder.

It includes:

* hardening rationale
* command-by-command execution narrative
* authorized vs refused access validation
* screenshots used as proof of work
* notes on RBAC limitations and custom role considerations

Suggested main file:

📘 `report/rapport_seance3_mongodb_style_seance2.pdf`

---

## 🖼️ Evidence Included

This repository is designed to document the work transparently.

Included evidence may contain:

* terminal outputs
* MongoDB command results
* authentication failure screenshots
* authenticated success screenshots
* report pages and proof snapshots

---

## 🚀 Possible Improvements

* Add custom MongoDB roles with collection-level permissions
* Add TLS configuration for encrypted transport
* Add Docker Compose for reproducible lab setup
* Add automated validation scripts for hardening checks
* Extend the lab with auditing and logging controls

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
