# Building a Public Key Infrastructure (PKI)
**Course:** CS 456 | **Date:** November 15, 2025

---

## Skills Demonstrated
- Designing and implementing a three-tier PKI hierarchy
- Certificate authority creation and certificate signing
- TLS/SSL certificate chain configuration
- Browser trust store management
- DNS resolution and host file manipulation

---

## Overview
This project involved building a functioning three-tier Public Key 
Infrastructure from scratch using OpenSSL. Starting from a self-signed 
Root CA, I manually signed each successive certificate authority down the 
chain, ultimately issuing an end-entity certificate to a local Flask web 
server. The end goal was a fully trusted HTTPS site at 
`https://ryan.andrews.edu:5000` that my browser could verify without warnings.

---

## Building the PKI

### Step 1: Creating the Root CA
I began by generating the Root CA — the foundation of the entire trust 
hierarchy. Using OpenSSL, I generated a private key and a self-signed 
certificate for the Root CA. Since the Root CA is the ultimate source of 
trust in the chain, I kept it isolated from the rest of the process, only 
bringing it into use when signing the Intermediate CA below it.

### Step 2: Creating and Signing the Intermediate CA
With the Root CA established, I generated a private key and certificate 
signing request (CSR) for an Intermediate CA. I then used the Root CA to 
manually sign the Intermediate CA's certificate, establishing it as a 
trusted authority beneath the root. This separation is intentional — it 
means the Root CA does not need to be involved in day-to-day certificate 
operations, reducing its exposure and risk of compromise.

### Step 3: Issuing the End-Entity Certificate
Using the same process, I generated a private key and CSR for the 
end-entity certificate assigned to `ryan.andrews.edu`. The Intermediate CA 
then signed this certificate, completing the three-tier chain: 
Root CA → Intermediate CA → End-Entity.

### Step 4: Assembling the Certificate Chain
I assembled a `fullchain.pem` file by bundling the end-entity certificate 
together with the Intermediate CA certificate. This is necessary because 
a browser verifying the server's certificate needs to be able to trace the 
full chain of trust back to the Root CA. The server certificate alone only 
proves it was signed — without the Intermediate CA certificate included, 
the browser cannot determine what signed it and will reject the connection.

### Step 5: Configuring Trust and DNS Resolution
Two final steps were required to make the site fully functional in the browser.

First, I imported `root-ca.crt` into Firefox's trust store. This tells 
Firefox to trust any certificate that chains back to this Root CA, 
establishing *who* to trust.

Second, I added an entry to `/etc/hosts` mapping `ryan.andrews.edu` to 
`127.0.0.1`. Since this is not a real publicly registered domain, this step 
was necessary to allow the browser to resolve the hostname to the local 
machine — establishing *where* to connect.

Without both steps in place, the site would either be unreachable or present 
an untrusted certificate warning. Together, they produced a fully functioning, 
warning-free HTTPS connection.

### Step 6: Deploying the Flask Server
The `fullchain.pem` and `ryan.andrews.edu.key` files were passed to a Flask 
web server configured to serve HTTPS, completing the project.

---

## Key Concepts

### Why the Root CA is Kept Offline
Keeping the Root CA offline dramatically reduces its attack surface. An 
offline system is not exposed to network-based vulnerabilities that could 
be exploited to compromise it. This is especially critical because every 
Intermediate and issuing CA in the hierarchy derives its trust from the 
Root CA — if it were compromised, the entire PKI would need to be rebuilt 
from scratch.

### The Advantage of a Tiered PKI
A three-tier hierarchy offers two key advantages over a single-tier model 
where the Root CA directly signs all certificates.

**Security through isolation:** If an Intermediate CA is compromised, it 
can be revoked and replaced without touching the Root CA — a far smaller 
remediation than rebuilding a root that directly signed hundreds of 
certificates.

**Manageability through delegation:** Intermediate CAs can be scoped to 
specific departments or teams within a large organization, each operating 
under the same root of trust. This allows certificate management to be 
delegated without ever granting access to the Root CA itself.
