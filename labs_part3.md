# Setting Up HashiCorp Vault for Certificates (PKI) with Docker

## ✨ What This Lab Is About

In this beginner-friendly lab, you'll:

* Start HashiCorp Vault using Docker
* Set up Vault as a Certificate Authority (CA)
* Create and issue a certificate using Vault
* Run a simple HTTPS web server that uses the certificate
* Understand what each command does and why

No prior experience with Vault or certificates is required. Let's get started!

---

## 🛠 Prerequisites

Before starting, make sure you have:

* **Docker** installed
* A **terminal** (Linux, macOS, or Windows WSL)
* Basic command-line familiarity

---

## 📁 Step 1: Run Vault in Docker

We use Docker to avoid installing Vault manually. Vault will run in **dev mode** (not for production!) so we don't need to worry about unsealing or configuring storage.

### Create `docker-compose.yml`

```yaml
version: '3.8'

services:
  vault:
    image: hashicorp/vault:latest
    container_name: vault
    ports:
      - "8200:8200"  # Expose Vault on localhost:8200
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: root  # Set the root token to 'root' for simplicity
      VAULT_DEV_LISTEN_ADDRESS: 0.0.0.0:8200  # Allow access from outside container
    cap_add:
      - IPC_LOCK  # Required by Vault
    command: "server -dev"  # Start Vault in development mode
```

### Start Vault

```bash
docker-compose up -d
```

### Why this order?

1. We define how Vault runs.
2. We expose port 8200 so we can talk to Vault.
3. We give it a known root token and tell it to run in dev mode (quick start).

---

## 🔐 Step 2: Login to Vault

Set the Vault address so the CLI knows where to connect:

```bash
export VAULT_ADDR=http://localhost:8200
```

Then login with the root token:

```bash
vault login root
```

### What happens here?

You authenticate as the admin (root) so you can configure Vault.

---

## 🔧 Step 3: Enable and Configure the PKI Secrets Engine

Vault has a **PKI secrets engine** that can generate and sign certificates.

### Enable PKI

```bash
vault secrets enable pki
```

> This activates the PKI system at the default path `/pki`.

### Set TTL (how long certs are valid)

```bash
vault secrets tune -max-lease-ttl=87600h pki
```

> `87600h` = 10 years. This is how long the root cert will be valid.

### Generate the Root Certificate

```bash
vault write pki/root/generate/internal \
    common_name="my.lab.internal" \
    ttl=87600h
```

> This creates a **self-signed root certificate** for the fake domain `my.lab.internal`.

### Set URLs for CRL and Issuing Cert

```bash
vault write pki/config/urls \
    issuing_certificates="$VAULT_ADDR/v1/pki/ca" \
    crl_distribution_points="$VAULT_ADDR/v1/pki/crl"
```

> These are metadata URLs that get embedded in issued certificates. They're not real here, but Vault requires them.

---

## 📄 Step 4: Create a Role to Allow Cert Issuing

Roles define **who can get what kind of cert**.

```bash
vault write pki/roles/webapp \
    allowed_domains="my.lab.internal" \
    allow_subdomains=true \
    max_ttl="72h"
```

> This role allows issuing certificates for names like `something.my.lab.internal`.

---

## 📅 Step 5: Generate a Certificate

```bash
vault write pki/issue/webapp \
    common_name="test.my.lab.internal" \
    ttl="24h"
```

> This asks Vault to issue a certificate for `test.my.lab.internal`.

Vault returns:

* `certificate`: the server certificate
* `private_key`: the private key for the cert
* `issuing_ca`: the root certificate

### Save these to files:

```bash
echo "<private_key>" > webapp.key
echo "<certificate>" > webapp.crt
echo "<issuing_ca>" > ca.crt
```

(Replace `<...>` with the actual values Vault returns)

---

## 🌐 Step 6: Run a Simple HTTPS Web Server

We will now run a basic HTTPS server to see the cert in action.

### Create a file `https_server.py`

```python
from http.server import HTTPServer, SimpleHTTPRequestHandler
import ssl

httpd = HTTPServer(('0.0.0.0', 4443), SimpleHTTPRequestHandler)
httpd.socket = ssl.wrap_socket(httpd.socket,
                               certfile="webapp.crt",
                               keyfile="webapp.key",
                               ca_certs="ca.crt",
                               server_side=True)
print("Serving on https://localhost:4443")
httpd.serve_forever()
```

### Run it:

```bash
python3 https_server.py
```

Open your browser at: [https://localhost:4443](https://localhost:4443)

You may see a warning (because your CA is not trusted by your OS).

---

## 🔍 Step 7: Test It with curl

```bash
curl -v --cacert ca.crt https://localhost:4443
```

You should see a successful connection:

* TLS handshake OK
* HTML output from the server

---

## 🌟 Summary: What You Just Did

| Step               | What You Achieved                          |
| ------------------ | ------------------------------------------ |
| Docker + Vault     | Quick Vault start without setup hassle     |
| Enabled PKI        | Vault can now be a Certificate Authority   |
| Issued cert        | Got a real cert and private key from Vault |
| Ran HTTPS server   | Used the cert in a working service         |
| Verified with curl | Proved the cert chain worked and was valid |

You now know how to use Vault to issue certificates and apply them to real services!
