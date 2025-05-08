# Part 3 - How TLS/SSL Works (With Handshake Diagram)

## 🔐 What Is the TLS Handshake?

The TLS handshake is how two systems (like your browser and a website) agree to talk securely.

---

## 🤝 Step-by-Step (Simple Version)

1. **Client:** "Hi! I want to talk securely. Here’s a random number."
2. **Server:** "Hi! Here’s my **certificate** and my public key."
3. **Client:** Checks the certificate:
   - Is it signed by a trusted CA?
   - Is it still valid?
   - Does it match the server's name?
4. **Client:** Encrypts a secret and sends it to the server.
5. **Server:** Unlocks the secret using its private key.
6. 🔐 Both now use the shared secret (session key) to talk securely.

---

## 🧠 What Happens Next?

Once the handshake is done:
- Communication is fast and secure
- Everything is encrypted using the session key

---

![Handshake](handshake.png)
