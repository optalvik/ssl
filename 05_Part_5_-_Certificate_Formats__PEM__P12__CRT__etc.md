# Part 5 - Certificate Formats (PEM, P12, CRT, etc.)

## 🔄 Part 3: **Certificate Formats & Replacing Old Certs**

---

### 📦 First: What Are Certificate File Formats?

Think of **certificate formats** like different *containers* for the same kind of info.

Imagine you want to carry your lunch: you could use a plastic box, a paper bag, or a thermos — same food, just packed differently.

---

### 🗂 Common Certificate File Formats

| Format          | What It Holds                                | Looks Like                            | Used For                         |
| --------------- | -------------------------------------------- | ------------------------------------- | -------------------------------- |
| `.pem`          | Certificate or key in plain text             | `-----BEGIN CERTIFICATE-----`         | Most common in Linux             |
| `.key`          | The **private key** (text)                   | `-----BEGIN PRIVATE KEY-----`         | Must stay secret                 |
| `.crt`          | Certificate file, same as `.pem`             | `-----BEGIN CERTIFICATE-----`         | Used on servers                  |
| `.pfx` / `.p12` | Cert + key + chain in **one encrypted file** | Binary                                | For Windows apps, Java, browsers |
| `.csr`          | Request for a certificate                    | `-----BEGIN CERTIFICATE REQUEST-----` | Sent to CA                       |
| `.der`          | Binary form of cert (not text)               | Binary                                | Used by Windows apps sometimes   |

---

### 🔑 Why Do Some Have Passwords?

Some formats (like `.p12`, `.pfx`) include:

* **Your private key**
* **Your certificate**
* **Maybe a chain of trust**

To protect all of that, they’re **encrypted with a password**.

Think of it like:

* A lunchbox **with a lock**
* You need the **password (key)** to open it

Applications like browsers, Java, or Windows servers will **ask for the password** when they try to use that certificate.

---

## 🛠️ Why We Use Them in Apps

| Use Case                                   | Format         | Why                          |
| ------------------------------------------ | -------------- | ---------------------------- |
| Linux apps (Nginx, Splunk, Elasticsearch)  | `.pem`, `.key` | Easy to read, common format  |
| Windows (IIS, Outlook)                     | `.pfx`, `.p12` | Bundled + password protected |
| Java apps (Kafka, Elasticsearch with Java) | `.p12` or JKS  | Java prefers this style      |
| Web browsers (for client auth certs)       | `.p12`         | Easy to import/export        |

---

## 🔁 What Happens When a Certificate Gets Old?

### Certificates Expire

Every certificate has a **“valid until”** date. When it passes, the certificate is no longer trusted.

You’ll see errors like:

* “This site’s certificate has expired!”
* “TLS handshake failed”
* “Could not verify peer”

---

## 🔄 Replacing an Expired Certificate (Like an ID Renewal)

### 🧭 Step-by-step:

1. **Create a new private key** (optional — unless rotating):

   ```bash
   openssl genrsa -out new.key 2048
   ```

2. **Generate a new CSR (Certificate Signing Request)**:

   ```bash
   openssl req -new -key new.key -out new.csr
   ```

3. **Send CSR to your CA** (internal or external)

4. **Receive the new certificate**

5. **Bundle it with any intermediates** (like before):

   ```bash
   cat new.crt intermediate.pem > new.bundle.pem
   ```

6. **Update your app’s config** to point to the new files

7. **Restart the app** so it uses the new certificate

---

## 😱 What if Your CA Gets Old or Expires?

Sometimes even your **CA (Certificate Authority)** expires (it’s rare, but it happens after \~10–20 years).

### In that case:

* You make a **new root CA**
* You generate **new certs** for every service
* You **distribute the new CA cert** to every machine or app
* Old certs signed by the old CA **stop being trusted**

---

## 🧠 TL;DR — Quick Facts

| Question                 | Answer                                            |
| ------------------------ | ------------------------------------------------- |
| What's in a `.pem` file? | Text cert or key                                  |
| Why use `.p12`?          | It’s a secure, zipped bundle (cert + key + chain) |
| Why passwords?           | To protect private keys inside `.p12`/`.pfx`      |
| How do we replace certs? | Generate → Sign → Replace → Restart               |
| What if CA expires?      | Replace CA + all certs it signed                  |
