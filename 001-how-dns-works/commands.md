# DNS Commands

This file contains useful Windows commands for investigating DNS resolution, DNS cache and network configuration.

---

## 1. `nslookup`

The `nslookup` command queries a DNS server to discover information about a domain.

```powershell
nslookup google.com
```

Example output:

```text
Server:  router.local
Address:  192.168.1.1

Non-authoritative answer:
Name:    google.com
Addresses:  2a00:1450:400b:c01::71
            142.250.179.78
```

### What to observe

* **Server**: the DNS server that answered the request.
* **Address**: the IP address of the DNS server.
* **Name**: the domain that was queried.
* **Addresses**: the IP addresses associated with the domain.

The response may contain:

* an IPv4 address;
* an IPv6 address;
* multiple IP addresses.

### Querying a specific DNS server

It is also possible to choose which DNS server will answer the query.

Google DNS:

```powershell
nslookup google.com 8.8.8.8
```

Cloudflare DNS:

```powershell
nslookup google.com 1.1.1.1
```

This is useful for comparing responses from different DNS resolvers.

---

## 2. `ipconfig /displaydns`

The `ipconfig /displaydns` command displays the DNS records currently stored in the Windows DNS cache.

```powershell
ipconfig /displaydns
```

Example:

```text
Record Name . . . . . : google.com
Record Type . . . . . : 1
Time To Live  . . . . : 240
Data Length . . . . . : 4
Section . . . . . . . : Answer
A (Host) Record . . . : 142.250.179.78
```

### What to observe

* **Record Name**: the domain stored in the cache.
* **Record Type**: the type of DNS record.
* **Time To Live**: how long the record can remain cached.
* **A Record**: the IPv4 address associated with the domain.

Windows stores these records temporarily to avoid repeating DNS queries every time the same domain is accessed.

---

## 3. `ipconfig /flushdns`

The `ipconfig /flushdns` command clears the Windows DNS resolver cache.

```powershell
ipconfig /flushdns
```

Expected output:

```text
Successfully flushed the DNS Resolver Cache.
```

After the cache is cleared, Windows must perform a new DNS query the next time a domain is accessed.

### When this command can be useful

* A website recently changed its IP address.
* Windows is using an outdated DNS record.
* A domain works on another device but not on the current computer.
* DNS resolution is returning incorrect information.

Clearing the DNS cache does not change the configured DNS server. It only removes the DNS records stored locally.

---

## Practical sequence

The following sequence can be used to observe DNS caching:

### Step 1 — Clear the cache

```powershell
ipconfig /flushdns
```

### Step 2 — Access or query a domain

```powershell
nslookup github.com
```

You can also open the domain in a browser.

### Step 3 — Inspect the cache

```powershell
ipconfig /displaydns
```

Search for records related to the domain.

---

## Important note

`nslookup` sends a direct query to a DNS server.

Because of this, running `nslookup` does not always create the same cache entries as accessing a domain through an application that uses the Windows DNS resolver.

A command such as:

```powershell
ping github.com
```

can also trigger Windows to resolve the domain name before attempting the network connection.

---

## Command summary

| Command                       | Purpose                                |
| ----------------------------- | -------------------------------------- |
| `nslookup domain.com`         | Queries DNS information about a domain |
| `nslookup domain.com 8.8.8.8` | Queries a specific DNS server          |
| `ipconfig /displaydns`        | Displays the Windows DNS cache         |
| `ipconfig /flushdns`          | Clears the Windows DNS cache           |
