# Part 8 - Certificate Lifecycle & Rotation

## 🔁 Part 4: **What Does "Rotating a Key" Mean?**

### 🧠 In Simple Words:

> **Rotating a key** means creating a **new private key** instead of using your old one.

### 🛠 Why Would You Rotate a Key?

1. 🔐 **Better security** – like changing the locks, not just repainting the door.
2. 🔄 **Regular updates** – some policies say: “Change keys every year.”
3. 😱 **Something went wrong** – if the old key might be stolen or shared by mistake.

---

### 🔑 Example:

```bash
# Step 1: Create a brand new private key
openssl genrsa -out new.key 2048

# Step 2: Make a new CSR using the new key
openssl req -new -key new.key -out new.csr
```

Then you send `new.csr` to the CA and get a **new certificate** tied to the **new key**.

🧠 So:

* If you don’t rotate: same key, new cert
* If you rotate: new key, new cert

---

### 🧰 Tools That Can Rotate Automatically:

| Tool                        | Rotates key + cert? | Notes                          |
| --------------------------- | ------------------- | ------------------------------ |
| **certbot (Let's Encrypt)** | ✅ Yes               | Every 90 days                  |
| **cert-manager**            | ✅ Yes               | Used in Kubernetes             |
| **Vault**                   | ✅ Yes               | Good for internal apps         |
| **Manual**                  | ❌ No                | You do it yourself via OpenSSL |
