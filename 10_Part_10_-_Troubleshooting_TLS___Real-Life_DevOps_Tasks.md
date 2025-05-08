# Part 10 - Troubleshooting TLS & Real-Life DevOps Tasks

## 🛠️ What DevOps and Sysadmins See Every Day

When something breaks, these are common problems:

---

### ❌ "Certificate Expired"

- The certificate wasn’t renewed in time.
- Fix: Get a new cert, update the server, and restart it.

---

### ❌ "Hostname Mismatch"

- The certificate is for `app.company.com`, but you connected to `api.company.com`.
- Fix: Get a cert with the correct **Common Name (CN)** or **Subject Alternative Name (SAN)**.

---

### ❌ "Untrusted Certificate Authority"

- The cert is signed by an internal CA that your system doesn’t trust.
- Fix: Install or trust the CA certificate manually.

---

### ❌ "Key Mismatch"

- The cert and private key on the server don’t match.
- Fix: Ensure you’re using the correct key for that cert.

---

## 🔧 Tools Used in Real Life

| Tool         | What It Does                                |
|--------------|---------------------------------------------|
| `openssl`    | Check certs, connect to servers manually     |
| `curl -v`    | See HTTPS problems when connecting to APIs   |
| Browser Dev Tools | View cert details (lock icon → view)    |

---

## 🧠 Summary

Troubleshooting is a **daily task** for DevOps and sysadmins — understanding certificates makes it way easier to solve real-world problems.
