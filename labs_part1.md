# 🔒 Enhanced Cybersecurity Lab for Kids: Web Security & Certificates

> **Welcome, Cyber Defenders!** In these labs, you'll learn how secure websites work, create your own digital certificates, and protect systems from cyber threats. Let's jump in!

## 👀 Common SSL/TLS Errors and What They Mean

Have you ever seen messages like these when visiting websites?

- "Your connection is not private"
- "Certificate not trusted"
- "Invalid certificate"

These are SSL/TLS errors! Today, we'll learn what they mean and how to fix them by understanding how certificates work.

## 📚 Understanding the Building Blocks

Before we begin, let's understand some key concepts with simple explanations:

| Concept | What It Is | Real-World Comparison |
|---------|------------|------------------------|
| **HTTPS** | Secure way websites talk to your browser | Like sending a letter in a locked box instead of a postcard |
| **Certificate** | Digital ID card that proves a website is real | Like a passport or school ID card |
| **Private Key** | Secret code only the owner knows | Your house key that should never be shared |
| **Certificate Authority (CA)** | Organization that issues certificates | Like the Department of Motor Vehicles that issues driver's licenses |
| **OpenSSL** | Tool to create and manage certificates | Like a digital locksmith's toolkit |

![Certificate Trust Chain](certi_trust.webp)

## 🧪 Lab 1: Secure Public Website with Certificate

### 🧠 Goal

Set up a public-style website using your own Certificate Authority. Learn what HTTPS really means and how certificates work.

### 🔧 Tools Needed

* One Linux VM (local or cloud)
* Ubuntu is recommended (20.04 or 22.04)
* OpenSSL (usually pre-installed)
* Python3

If you don't have OpenSSL:

```bash
sudo apt update
sudo apt install openssl
```

> 💡 **Did You Know?** OpenSSL is one of the most used security tools in the world! Most secure websites rely on it to keep your information safe.

---

### 🤔 What Actually Happens When You Visit a HTTPS Website?

1. Your browser says "Hello" to the website
2. The website sends its certificate (ID card)
3. Your browser checks if the certificate is signed by a trusted CA
4. If trusted, your browser creates a secure connection
5. Now all data sent between you and the website is encrypted (scrambled so others can't read it)

---

### 🛠️ Detailed OpenSSL Command Guide

#### 🔑 `openssl genrsa -out file.key 2048`

* **What it does:** Creates a new private key using RSA encryption
* **Command breakdown:**
  * `genrsa` - Tells OpenSSL to generate an RSA key pair
  * `-out file.key` - Saves the key to this file
  * `2048` - The key size in bits (bigger = stronger)
* **Think of it as:** Creating a super-complex secret password that only you know
* **Why it matters:** This private key is the foundation of your security - keep it safe!

#### 🪪 `openssl req -x509 ...`

* **What it does:** Creates a self-signed certificate (makes you your own CA)
* **Command breakdown:**
  * `req` - Request a new certificate
  * `-x509` - Make it a self-signed certificate
  * `-new` - Create a new request
  * `-nodes` - No encryption on the private key file (so servers can use it without password)
  * `-key file.key` - Use this private key
  * `-sha256` - Use this secure hashing algorithm (mathematical formula)
  * `-days 3650` - Certificate will be valid for 10 years
  * `-out file.crt` - Save the certificate to this file
  * `-subj "/CN=Name"` - Set the Common Name
* **Think of it as:** Creating your own official ID card maker, then making an ID card for yourself
* **Why it matters:** Now you can create certificates that you control!

#### 📮 `openssl req -new -newkey ...`

* **What it does:** Creates both a new private key AND a certificate signing request (CSR)
* **Command breakdown:**
  * `req` - Certificate request
  * `-new` - Make a new request
  * `-newkey rsa:2048` - Also generate a new 2048-bit RSA key
  * `-nodes` - No password encryption on the key
  * `-keyout file.key` - Save the key here
  * `-out file.csr` - Save the request here
  * `-subj "/CN=..."` - Set the Common Name (usually domain name)
* **Think of it as:** Creating a key and filling out an application form for an ID card
* **Why it matters:** This is how websites request trusted certificates

#### ✅ `openssl x509 -req ...`

* **What it does:** Signs a CSR with your CA to create a valid certificate
* **Command breakdown:**
  * `x509` - Work with certificates
  * `-req` - Input is a certificate request
  * `-in file.csr` - The request file
  * `-CA ca.crt` - Your CA certificate
  * `-CAkey ca.key` - Your CA's private key
  * `-CAcreateserial` - Create/update a serial number file
  * `-out file.crt` - Save the signed certificate here
  * `-days 365` - Valid for 1 year
* **Think of it as:** The Department of ID Cards approving an application and issuing an official ID
* **Why it matters:** This turns a request into a trusted certificate

---

### 🔍 Advanced Certificate Inspector

```bash
openssl x509 -in certificate.crt -text -noout
```

* **What it does:** Shows all the details inside a certificate
* **Command breakdown:**
  * `x509` - Work with certificates
  * `-in certificate.crt` - The certificate to examine
  * `-text` - Show in human-readable format
  * `-noout` - Don't output the encoded version
* **Think of it as:** Using a magnifying glass to read all the fine print on an ID card
* **Why it matters:** You can verify all certificate details, like who issued it and when it expires

---

### 🛠️ Instructions

#### What are we building?

A fun and secure mini-website using encryption (HTTPS). You'll play the role of a **Certificate Authority**, and make your browser trust your website.

#### 1. Prepare your working directories

```bash
mkdir -p ~/tls-lab/public_ca ~/tls-lab/public_site
cd ~/tls-lab
```

**What this does:** Creates two folders:
* `public_ca` = your pretend certificate authority
* `public_site` = your mini website files

**Think of it as:** Setting up an office for your ID card company and a separate building for your website

#### 2. Create your own public Certificate Authority (CA)

```bash
# Step 1: Create a super-secret key for your certificate authority
openssl genrsa -out public_ca/public_ca.key 2048
```
**What just happened:** You created a private key (2048 bits strong) that your CA will use to sign certificates.

```bash
# Step 2: Create your CA certificate (self-signed)
openssl req -x509 -new -nodes -key public_ca/public_ca.key -sha256 -days 3650 -out public_ca/public_ca.crt -subj "/CN=MyPublicRootCA"
```
**What just happened:** You created a self-signed certificate, making yourself a Certificate Authority. This certificate is valid for 10 years (3650 days).

> 🔍 **Let's explore!** Run the command below to look inside your CA certificate:
> ```bash
> openssl x509 -in public_ca/public_ca.crt -text -noout | less
> ```
> Look for information like:
> - When it was created
> - When it expires
> - The "issuer" and "subject" (both should be your CA)

#### 3. Create a non-privileged user for the web service

```bash
sudo useradd -r -s /usr/sbin/nologin kidweb
```

**What this does:** Creates a restricted user account that:
* Has no password
* Cannot log in
* Has minimal permissions
* Will only run your website

**Why it matters:** This is a security best practice called "least privilege" - give programs only the access they need and nothing more.

#### 4. Generate key and certificate for your public website

```bash
# Step 1: Create a key and certificate request
openssl req -new -newkey rsa:2048 -nodes -keyout public_site/website.key -out public_site/website.csr -subj "/CN=kids-website.local"
```
**What just happened:** You created:
1. A new private key for your website (`website.key`)
2. A Certificate Signing Request (`website.csr`) - like an application form for a certificate

```bash
# Step 2: Sign the certificate with your CA
openssl x509 -req -in public_site/website.csr -CA public_ca/public_ca.crt -CAkey public_ca/public_ca.key -CAcreateserial -out public_site/website.crt -days 365
```
**What just happened:** Your CA used its private key to sign the website's certificate request, creating a valid certificate good for 1 year.

> 💡 **Certificate Chain:** You've created a chain of trust:
> Browser → trusts your CA → which signed your website's certificate

#### 5. Create a fun website

```bash
echo "<html><body><h1>🌐 Hello from your secure kids website!</h1><p>The secret code is: CYBERKID2025</p></body></html>" > public_site/index.html
```

**What this does:** Creates a simple webpage with HTML code

**Try it yourself:** Modify the HTML to add your own message or even some colors!

#### 6. Set proper security permissions

```bash
chmod 600 public_site/website.key
chown kidweb:kidweb public_site/website.key
```

**What this does:**
* `chmod 600` - Only the file owner can read/write this file
* `chown kidweb:kidweb` - Changes the owner to the kidweb user

**Why it matters:** Protects your private key from being accessed by other users or programs

#### 7. Run your secure site as kidweb user

```bash
cd public_site
sudo -u kidweb python3 -m http.server 4433 --bind 127.0.0.1 --directory . \
  --ssl-certfile website.crt --ssl-keyfile website.key
```

**What this does:**
* `sudo -u kidweb` - Runs the command as the kidweb user
* `python3 -m http.server` - Starts a simple web server
* `4433` - The port number to use
* `--bind 127.0.0.1` - Only accessible from your computer
* `--ssl-certfile` and `--ssl-keyfile` - Enables HTTPS with your certificate and key

➡️ Open your browser and visit [https://127.0.0.1:4433](https://127.0.0.1:4433)

You'll probably see a warning because your browser doesn't trust your CA yet!

---

### 🧠 How to Trust Your CA in Firefox or Chrome

1. Open your browser
2. Go to **Settings > Privacy & Security > Certificates**
3. Click **View Certificates** or **Import**
4. Choose `public_ca/public_ca.crt`
5. Mark it as a **Trusted Root Authority**

✅ Now your browser won't show warnings when visiting your site!

---

### 🔍 Certificate Verification Challenge

Try this to verify your certificate is properly signed:

```bash
openssl verify -CAfile public_ca/public_ca.crt public_site/website.crt
```

If you see `public_site/website.crt: OK`, your certificate is valid!

---

## 🧪 Lab 2: Internal API with Internal Certificate

### 🧠 Goal

Simulate an **internal service** like a backend API or a database. It needs its own secure certificate.

🔍 **What is an API?**
API = "Application Programming Interface." It's how programs talk to each other — like a waiter taking your order to the kitchen.

Example: A weather app asks an API, "What's the weather in Tallinn?" — the API answers securely.

---

### 🛠️ Steps

#### 1. Create a new internal CA

```bash
# Create folders for our internal system
mkdir -p ~/tls-lab/internal_ca ~/tls-lab/internal_api
cd ~/tls-lab

# Generate a new private key for internal CA
openssl genrsa -out internal_ca/internal_ca.key 2048
```
**What this does:** Creates a new 2048-bit RSA private key for your internal Certificate Authority

```bash
# Create the internal CA certificate
openssl req -x509 -new -nodes -key internal_ca/internal_ca.key -sha256 -days 3650 -out internal_ca/internal_ca.crt -subj "/CN=InternalCA"
```
**What this does:** Creates a self-signed certificate for your internal CA, valid for 10 years

**Why have a separate CA?** Security best practice! Internal services should use different certificates than public-facing ones.

#### 2. Create a user to run your API

```bash
sudo useradd -r -s /usr/sbin/nologin kidapi
```

**What this does:** Creates another restricted user just for running your API service

**Why it matters:** Separation of duties - different services run as different users for better security

#### 3. Create a cert for the internal API

```bash
# Generate key and certificate request
openssl req -new -newkey rsa:2048 -nodes -keyout internal_api/api.key -out internal_api/api.csr -subj "/CN=internal.api"
```
**What this does:** Creates a private key and certificate request for your API service

```bash
# Sign the certificate with your internal CA
openssl x509 -req -in internal_api/api.csr -CA internal_ca/internal_ca.crt -CAkey internal_ca/internal_ca.key -CAcreateserial -out internal_api/api.crt -days 365
```
**What this does:** Your internal CA signs the API's certificate request, creating a valid certificate

#### 4. Add a response page

```bash
echo "<html><body><h1>🔒 Internal API: Hello, secure world!</h1><p>Weather data: {\"city\":\"Tallinn\",\"temp\":\"15°C\",\"condition\":\"Partly Cloudy\"}</p></body></html>" > internal_api/index.html
```

**What this does:** Creates a simple API response page with some pretend weather data in JSON format

#### 5. Set secure permissions

```bash
chmod 600 internal_api/api.key
chown kidapi:kidapi internal_api/api.key
```

**What this does:** Restricts access to the API's private key so only the kidapi user can read it

#### 6. Run the API server

```bash
cd internal_api
sudo -u kidapi python3 -m http.server 8443 --bind 127.0.0.1 --directory . \
  --ssl-certfile api.crt --ssl-keyfile api.key
```

**What this does:** Starts a secure API server running as the kidapi user on port 8443

➡️ It's now running on [https://127.0.0.1:8443](https://127.0.0.1:8443)

#### 7. Test the API using curl

```bash
curl --cacert ../internal_ca/internal_ca.crt https://127.0.0.1:8443
```

**What this does:**
* `curl` - A tool to make web requests
* `--cacert` - Use this CA certificate to verify the server
* The URL of your API

**Think of it as:** One program (curl) talking securely to another program (your API)

---

### 🔍 Fingerprint Explorer

Each certificate has a unique "fingerprint" (like a digital thumbprint). Let's see yours:

```bash
# View your website certificate's fingerprint
openssl x509 -in public_site/website.crt -noout -fingerprint

# View your API certificate's fingerprint 
openssl x509 -in internal_api/api.crt -noout -fingerprint
```

**What this does:** Shows the SHA1 fingerprint of each certificate

**Why it matters:** Fingerprints can be used to verify a certificate's identity quickly

---

## 🚨 Lab 3: Security Incident — Key Leaked!

### 🧠 Goal

A bad actor got your private key! You need to **rotate it** (replace it) ASAP.

---

### 🔄 Steps to Recover

#### 1. Make a new key

```bash
# Generate a new private key
openssl genrsa -out internal_api/api_rotated.key 2048
```

**What this does:** Creates a brand new private key to replace the compromised one

**Think of it as:** Changing the locks on your door after someone stole your key

#### 2. Make a new cert request

```bash
# Create a new certificate request with the new key
openssl req -new -key internal_api/api_rotated.key -out internal_api/api_rotated.csr -subj "/CN=internal.api"
```

**What this does:** Creates a new Certificate Signing Request (CSR) using your new private key

**Think of it as:** Filling out paperwork for a replacement ID

#### 3. Sign a new certificate

```bash
# Sign the new certificate request
openssl x509 -req -in internal_api/api_rotated.csr -CA internal_ca/internal_ca.crt -CAkey internal_ca/internal_ca.key -CAcreateserial -out internal_api/api_rotated.crt -days 365
```

**What this does:** Your CA signs the new request, creating a valid replacement certificate

**Why not revoke the old one?** In a real-world scenario, you would also create a Certificate Revocation List (CRL) to tell everyone the old certificate shouldn't be trusted anymore.

#### 4. Swap in the new files

```bash
# Replace the old files with the new ones
cp internal_api/api_rotated.key internal_api/api.key
cp internal_api/api_rotated.crt internal_api/api.crt
```

**What this does:** Replaces the compromised files with your new secure ones

#### 5. Update permissions

```bash
# Set proper permissions on the new key
chmod 600 internal_api/api.key
chown kidapi:kidapi internal_api/api.key
```

#### 6. Restart the API server

```bash
# Stop the current server (press Ctrl+C in its terminal)
# Then restart with new certificate/key
sudo -u kidapi python3 -m http.server 8443 --bind 127.0.0.1 --directory . \
  --ssl-certfile api.crt --ssl-keyfile api.key
```

✅ Done! Your system is secure again. Great job acting like a real incident responder!

---

## 🎮 Bonus Challenges

Try these extra activities to level up your skills:

### 1. Certificate Revocation List (CRL)

```bash
# Create a revocation list for your internal CA
openssl ca -gencrl -keyfile internal_ca/internal_ca.key -cert internal_ca/internal_ca.crt -out internal_ca/internal_ca.crl
```

**What this does:** Creates a Certificate Revocation List - a blacklist of certificates that should no longer be trusted

**Think of it as:** A "Wanted" poster for compromised certificates

### 2. Certificate with Multiple Hostnames

```bash
# Create a config file for multiple names
echo "[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
[req_distinguished_name]
[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = kids-website.local
DNS.2 = www.kids-website.local
DNS.3 = admin.kids-website.local" > multi_domain.cnf

# Create a certificate with multiple domains
openssl req -new -newkey rsa:2048 -nodes -keyout multi_site.key -out multi_site.csr -subj "/CN=kids-website.local" -config multi_domain.cnf

# Sign it with your CA
openssl x509 -req -in multi_site.csr -CA public_ca/public_ca.crt -CAkey public_ca/public_ca.key -CAcreateserial -out multi_site.crt -days 365 -extensions v3_req -extfile multi_domain.cnf
```

**What this does:** Creates a certificate that works for multiple domain names

**Think of it as:** One ID card that works at multiple locations

### 3. Secret Message Challenge

Create an encrypted message only your friend can read:

```bash
# Create a secret message
echo "Meet me at the playground at 3pm for ice cream!" > secret.txt

# Encrypt it with your friend's certificate
openssl rsautl -encrypt -inkey public_site/website.crt -certin -in secret.txt -out secret.enc
```

Then have your friend decrypt it:

```bash
# Decrypt the message with the private key
openssl rsautl -decrypt -inkey public_site/website.key -in secret.enc -out secret_decrypted.txt

# Read the message
cat secret_decrypted.txt
```

---

## 🏆 Cybersecurity Badges

Congratulations! You've earned these badges:

- 🛡️ **Certificate Authority Creator**
- 🔒 **HTTPS Website Builder**
- 🔐 **API Security Specialist**
- 🚨 **Incident Response Handler**

---

## 📚 Real-World Applications

What you've learned is used every day in:

- 🌐 **Secure Websites** - Every HTTPS site uses certificates like these
- 📱 **Mobile Apps** - Many apps use certificates to talk securely to servers
- 💼 **Business Networks** - Companies use internal CAs for their services
- 🏦 **Banking Systems** - Financial institutions rely on certificates for security

## ☕ Bonus Lab 4: Working with Java and Certificates

Many applications are written in Java, and Java has its own way of handling certificates. Let's explore how to make a Java application trust our certificates!

### 🧠 Understanding Java's Certificate System

Java uses a file called a **truststore** to keep track of which certificates it trusts. This is similar to how your browser maintains a list of trusted certificates.

🔍 **What is a Java truststore?**
- It's a special file (usually called `cacerts`) that contains trusted certificates
- Java applications check this file when making secure connections
- By default, it includes certificates from major public Certificate Authorities

### Common Java SSL Errors

If a Java application cannot verify a certificate, you might see errors like:
- `PKIX path building failed`
- `SSLHandshakeException: unable to find valid certification path to requested target`

These errors mean Java doesn't trust the certificate that a website or API is presenting.

### 🔍 Certificate Chains Explained

When a website presents its certificate, it's often part of a chain:

```
Root CA Certificate (most trusted)
     ↓
Intermediate Certificate(s)
     ↓
Website/Server Certificate (leaf)
```

Java needs to trust either the specific certificate or (more commonly) the Root CA certificate that signed it.

### 🛠️ Adding Our Certificates to Java's Truststore

Let's make a Java application trust our certificates:

```bash
# First, find Java's default truststore location
# It's typically at:
# $JAVA_HOME/lib/security/cacerts
# Default password is usually "changeit"

# Add your CA certificate to Java's truststore
keytool -import -alias my-public-ca -file ~/tls-lab/public_ca/public_ca.crt -keystore $JAVA_HOME/lib/security/cacerts

# When prompted, enter the truststore password (default is "changeit")
```

**What this does:**
- `keytool` - Java's tool for managing certificates
- `-import` - Add a certificate to the truststore
- `-alias my-public-ca` - Give your certificate a friendly name
- `-file ~/tls-lab/public_ca/public_ca.crt` - The certificate to import
- `-keystore $JAVA_HOME/lib/security/cacerts` - Location of Java's truststore

### 🔍 Viewing Certificates in the Truststore

You can see what certificates Java trusts with:

```bash
keytool -list -keystore $JAVA_HOME/lib/security/cacerts
```

### 📱 Creating a Simple Java HTTPS Client

Let's create a simple Java program that connects to our secure website:

```bash
# Create a directory for our Java example
mkdir -p ~/tls-lab/java-example
cd ~/tls-lab/java-example

# Create a Java file
cat > HttpsClient.java << 'EOF'
import java.net.URL;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import javax.net.ssl.HttpsURLConnection;

public class HttpsClient {
    public static void main(String[] args) {
        try {
            // The URL of our secure website
            URL url = new URL("https://127.0.0.1:4433");
            
            // Open connection
            HttpsURLConnection connection = (HttpsURLConnection) url.openConnection();
            
            // Print response code
            System.out.println("Response Code: " + connection.getResponseCode());
            
            // Read response
            BufferedReader in = new BufferedReader(
                new InputStreamReader(connection.getInputStream()));
            String inputLine;
            StringBuffer response = new StringBuffer();
            while ((inputLine = in.readLine()) != null) {
                response.append(inputLine);
            }
            in.close();
            
            // Print response
            System.out.println("Response: " + response.toString());
            
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
EOF

# Compile the Java file
javac HttpsClient.java

# Run the Java program
java HttpsClient
```

**What this does:**
- Creates a simple Java program that connects to our secure website
- Prints the response code and content
- If you've added your CA to Java's truststore, it should work without errors!

### 🔐 Best Practices for Certificate Management in Java

1. **Always trust the root CA certificate** rather than individual server certificates
   - This way, when server certificates are renewed, you don't need to update your truststore

2. **Use meaningful aliases** when adding certificates
   - Makes it easier to identify and manage certificates later

3. **On Linux systems, use system tools** rather than manually updating the truststore
   - RHEL/CentOS/Fedora: Use `update-ca-trust`
   - Ubuntu: Use `update-ca-certificates`

---

## 🔍 Digging Deeper

Want to learn more? Here are some topics to explore:

- **Certificate Transparency**: How the internet keeps track of all certificates
- **HSTS**: Telling browsers to always use HTTPS
- **Let's Encrypt**: A free CA for the public internet
- **mTLS**: When both sides need certificates (mutual TLS)

---

Happy learning, Cyber Defenders! Remember - with great knowledge comes great responsibility. Use your powers for good! 🦸‍♂️🦸‍♀️