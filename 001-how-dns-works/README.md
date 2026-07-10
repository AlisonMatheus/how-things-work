# 🌐 How DNS Works

> Understanding how the Internet translates names into IP addresses.

## 📖 Introduction

Imagine if every website on the Internet could only be accessed by numbers.

Instead of typing:

google.com

you would need to remember something like:

142.250.74.206

That would be almost impossible.

The Domain Name System (DNS) solves this problem by translating human-friendly domain names into IP addresses that computers can understand.

---

# 🧩 The Problem

Humans are good at remembering names.

Computers only understand IP addresses.

Without DNS:

❌ Users would have to memorize thousands of IP addresses.

With DNS:

✅ Users only need to remember domain names.

---

# 💡 The Solution

DNS works like the phone book of the Internet.

Instead of asking:

"Where is Google?"

Your computer asks:

"What is the IP address for google.com?"

DNS replies:

google.com → 142.250.xxx.xxx

Your computer can now communicate directly with Google's server.

---

# 🔄 DNS Resolution

```text
User

    │
    ▼

google.com

    │
    ▼

DNS Resolver

    │
    ▼

142.250.xxx.xxx

    │
    ▼

Google Server

    │
    ▼

Website
```

---

# 📦 DNS Cache

Looking up DNS every time would be slow.

To improve performance, the operating system stores DNS responses in cache.

Example:

First request:

google.com

↓

Ask DNS

↓

Receive IP

↓

Store in cache

Second request:

google.com

↓

Read from cache

↓

No DNS query needed

---

# ⏱️ TTL (Time To Live)

Every DNS response includes a TTL.

Example:

google.com

↓

142.250.xxx.xxx

↓

TTL = 300 seconds

After the TTL expires, the computer performs a new DNS lookup.

---

# 🧠 Analogy

DNS is like searching for a contact in your phone.

You remember:

"Mom"

Your phone knows:

+55 11 99999-9999

DNS works exactly the same way.

You remember:

google.com

The computer uses:

142.250.xxx.xxx

---

# ❌ Common Misconceptions

## DNS opens websites.

No.

DNS only translates names into IP addresses.

Protocols like TCP, TLS and HTTP handle the communication after DNS finishes its job.

---

## DNS is queried every time.

Not necessarily.

If the information is still cached and the TTL hasn't expired, the computer uses the cached IP.

---

# 🛠️ Practical Experiment

Run:

```bash
nslookup google.com

nslookup github.com

```

Observe:

- IP Address
- DNS Server
- Response

---

# 📚 Key Concepts

- Domain Name
- IP Address
- DNS
- DNS Cache
- TTL
- Resolver

---

# 🎯 What I Learned

- Why DNS exists.
- Why computers need IP addresses.
- How DNS translates domain names.
- Why cache improves performance.
- What TTL is.
- Why DNS is only responsible for name resolution.

---
