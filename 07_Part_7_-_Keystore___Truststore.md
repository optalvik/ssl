# Part 7 - Keystore & Truststore

## 🗄️ Part 6: **What Are Keystore and Truststore?**

*(And why are they not the same?)*

### 💡 Imagine:

You run a secure messaging app. You want:

1. Your app to **prove who it is**
2. Your app to **know who to trust**

That's exactly what **keystores** and **truststores** are for.

---

### 🔑 What is a Keystore?

A **keystore** is where your application keeps **its own identity** — like its *private key* and *certificate*.

🧠 Think:

> "This is **me**, and here’s my proof (my certificate) signed by someone trusted."

#### Contains:

* 🔐 **Private key**
* 📄 **Certificate** (issued to you)
* (Possibly) a **chain of trust**

#### File types:

* `.jks` (Java Keystore — Java specific)
* `.p12` / `.pfx` (PKCS#12 — more universal)

#### Example:

Used by a Java app or Elasticsearch node to **prove it is a trusted service**.

---

### 🛡️ What is a Truststore?

A **truststore** is where your app keeps a list of **who it trusts** — certificates from **CAs or servers** it believes are real.

🧠 Think:

> "These are the **people I trust**. If someone gives me a certificate, I’ll check if it’s signed by one of them."

#### Contains:

* 🌍 **Public CA certificates**
* 🏢 **Internal CA certificates**
* 🧑 Maybe individual **peer certs** (in strict setups)

#### File types:

* `.jks` (Java)
* `.pem` or `.crt` in Linux

---

### 🎓 Analogy for Teens:

| Real Life Thing               | Keystore           | Truststore |
| ----------------------------- | ------------------ | ---------- |
| Your Student ID               | Proves who you are | ✔️         |
| The List of Approved Teachers | Who you trust      | ✔️         |

So:
🗝️ **Keystore = your ID**
✅ **Truststore = your trust list**

---

### ⚙️ In Real Systems:

| App                                   | Uses Keystore?                | Uses Truststore?                     |
| ------------------------------------- | ----------------------------- | ------------------------------------ |
| Web server (e.g., Nginx, Apache)      | ✅ yes                         | ✅ yes                                |
| Java app (e.g., Elasticsearch, Kafka) | ✅ yes (with `.jks` or `.p12`) | ✅ yes (to validate others)           |
| Browser                               | ❌ no personal key (usually)   | ✅ yes — has trust list of public CAs |

---

### 🔄 Can You Combine Them?

Yes — in formats like `.p12`, you can put **private key + cert + trusted CAs** all in one file, but logically they still do different jobs.