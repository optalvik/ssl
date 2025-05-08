# Part 9 - Automation Tools

## 🤖 Part 7: **Certificate Automation — Tools, How They Work, and Differences**

As systems grow, managing certs by hand becomes messy and risky. So we use **automation tools** to handle everything for us — creating, renewing, rotating, and installing certificates.

---

### 🧠 Why Automate Certificates?

| Problem             | Automation Solves It               |
| ------------------- | ---------------------------------- |
| Certs expire        | Automatically renews before expiry |
| Manual errors       | Scripts reduce mistakes            |
| Too many services   | Scales across 100s of apps         |
| Secure environments | Handles keys without exposing them |

---

### 🔧 Common Automation Tools (Simple Table)

| Tool                     | Use Case                         | Handles Key + Cert?    | For Internal or Public?  |
| ------------------------ | -------------------------------- | ---------------------- | ------------------------ |
| **Certbot**              | Websites (Let’s Encrypt)         | ✅ Yes                  | 🌍 Public                |
| **cert-manager**         | Kubernetes clusters              | ✅ Yes                  | 🏠 Internal or 🌍 Public |
| **Vault (by HashiCorp)** | Microservices, secrets, PKI      | ✅ Yes                  | 🏠 Internal              |
| **Smallstep CA**         | Lightweight internal CA          | ✅ Yes                  | 🏠 Internal              |
| **Ansible / Puppet**     | Configuration management         | ⚠️ Partially (depends) | 🏠 Internal              |
| **ACME protocol**        | Used by Let’s Encrypt and others | ⚙️ Protocol only       | Used by other tools      |

---

### 🛠 How They Work (At a Glance)

#### 🔁 Certbot (with Let’s Encrypt)

1. Runs every day as a cron job
2. If cert is near expiry, generates a new CSR
3. Gets cert via ACME
4. Installs it and reloads your web server

#### 🧬 cert-manager (Kubernetes)

1. Watches for certs in Kubernetes
2. Talks to CA (via ACME or webhook)
3. Automatically renews and mounts certs into pods

#### 🔐 Vault (with PKI engine)

1. Apps ask Vault: “Give me a cert for service1”
2. Vault signs it on the fly
3. Cert is short-lived (hours or days)
4. Apps use it, then get a new one later

#### 🏗 Smallstep CA

1. Lightweight CA that can issue certs using ACME or OAuth
2. CLI or API for scripting cert requests
3. Great for internal setups

---

### 📏 How Many Systems Use Each?

| Tool               | Popularity                                         |
| ------------------ | -------------------------------------------------- |
| **Certbot**        | 🔥 Super common on public websites                 |
| **cert-manager**   | 🔥 Standard in Kubernetes                          |
| **Vault**          | 🧠 Common in secure or microservice-heavy orgs     |
| **Smallstep**      | 💡 Lightweight alternative to Vault                |
| **Ansible/Puppet** | Used when infra config is already managed this way |

---

### 🧠 Summary: Which Tool for What?

| Scenario                      | Recommended Tool                        |
| ----------------------------- | --------------------------------------- |
| Public website with HTTPS     | **Certbot**                             |
| Internal apps on Kubernetes   | **cert-manager**                        |
| High-security microservices   | **Vault**                               |
| Simple internal CA            | **Smallstep CA**                        |
| Configuration-managed servers | **Ansible/Puppet with OpenSSL scripts** |
