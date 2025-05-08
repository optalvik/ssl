# Part 6 - Internal vs External Certificates

Just like there are different types of ID cards in real life, there are different types of certificates too!

### 1. Public Certificates (External Certificates)

| Feature    | Description                                  |
| ---------- | -------------------------------------------- |
| Issued by  | Public Certificate Authorities like Let's Encrypt |
| Trusted by | All browsers and computers automatically     |
| Used for   | Public websites like YouTube or Google       |

**Real-Life Comparison:** These are like **country passports** that work anywhere in the world!

**Fun Fact:** Your browser already knows and trusts about 100 certificate authorities that can issue these passports!

### 2. Private Certificates (Internal Certificates)

| Feature    | Description                                  |
| ---------- | -------------------------------------------- |
| Issued by  | A school or company's own certificate system |
| Trusted by | Only computers that are specially configured |
| Used for   | Private school or company systems            |

**Real-Life Comparison:** These are like **school ID cards** that only work at your school!

### 3. Self-Signed Certificates

| Feature    | Description                                  |
| ---------- | -------------------------------------------- |
| Issued by  | The computer itself                          |
| Trusted by | Nobody, unless specially added to trusted list |
| Used for   | Testing, development, or private use         |

**Real-Life Comparison:** These are like **homemade ID cards** you made yourself with markers and paper!

**Think About It:** When might someone trust a homemade ID card? (Hint: maybe in a club you made with friends?)

---

Certificates come in many different varieties for different purposes!

### Domain Validation (DV) Certificates
- The simplest type
- Only checks: "Do you own this website?"
- Like a basic library card

### Organization Validation (OV) Certificates
- More secure - checks the organization is real
- Like a school ID with both your name AND school name

### Extended Validation (EV) Certificates
- Super-duper secure!
- Does lots of background checks
- Like a passport with all the special security features
- Used by banks and stores where security is very important

### Special-Purpose Certificates:

**Wildcard Certificates**: Work for many similar websites (*.example.com)
- Like one key that opens all doors in your house

**Multi-Domain Certificates**: Work for different website names
- Like one ID card that works at school, library, AND swimming pool

**Code Signing Certificates**: For computer programs
- Like a "safety tested" sticker on toys

---

## What's Inside a Certificate?

Every certificate has important parts, just like ID cards have different information on them!

### The Certificate Parts:

1. **Subject Name** - Who the certificate belongs to
2. **Expiration Date** - When it stops working
3. **Public Key** - A special mathematical code used for security
4. **Digital Signature** - Proves who issued it
5. **Issuer Name** - Who created the certificate

**Activity:** Next time you're on a website, click the lock icon in your browser, then select "Certificate" to see these parts!

---

## Why Don't All Websites Use the Same Type?

Great question! Let's explore why we have different types:

### Public Certificates:
- ✅ Work everywhere automatically
- ✅ Trusted by all browsers
- ❌ Cost money
- ❌ Need to prove you own the website

### Internal Certificates:
- ✅ Free to create
- ✅ Complete control
- ❌ Only work within your organization
- ❌ Need special setup

### Self-Signed:
- ✅ Super easy to create
- ✅ Free
- ❌ Not trusted by any browser
- ❌ Cause warning messages

**Thought Experiment:** If you made a website for your school project, which would you use? What about if you made a website for a million people to use?

---

## Warning Messages You Might See

Your browser checks certificates to keep you safe! Here are warnings you might see:

### Warning: "Your connection is not private"
This happens when:
- The certificate has expired
- The certificate is from somewhere your browser doesn't trust
- The certificate is being used for the wrong website

**What to do:** This is like a security guard saying "This ID looks suspicious!" Be careful - this site might not be safe!

---

## 📚 Real-World Examples

Let's see how certificates work in real life:

### Example 1: Shopping Online
When you visit an online store:
1. Your browser checks the store's certificate
2. If it's trusted, you see the lock 🔒 icon
3. This means your payment information will be kept secret

### Example 2: School Computers
At school, computers might use internal certificates to:
1. Connect to the school's systems
2. Keep student information private
3. Make sure only school computers can access certain files

### Example 3: Making Your Own Website
When learning to code and make websites:
1. You might use a self-signed certificate for testing
2. When your site is ready for the world, you'd get a public certificate
3. Then everyone can visit safely!

---

## 📚 Activity - Certificate Detective!

Now let's practice being certificate detectives! Here's what to do:

1. Visit these websites:
   - Your school's website
   - A news website
   - A big online store

2. For each site, click the lock icon 🔒 and check:
   - Who issued the certificate?
   - When does it expire?
   - What type of certificate is it?

3. **Bonus challenge:** Can you find a website that doesn't have a valid certificate? (It would show a warning or no lock icon)