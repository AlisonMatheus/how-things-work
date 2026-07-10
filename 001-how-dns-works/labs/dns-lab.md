# DNS Laboratory

This document contains practical experiments performed while learning how DNS works.

---

# Lab 1 — Resolve a Domain Name

## Objective

Verify that DNS translates a domain name into an IP address.

## Command

```powershell
nslookup google.com
```

## Result

```text
Server: 192.168.8.1

Name: google.com

Addresses:  2a00:1450:400b:c02::64
          2a00:1450:400b:c02::8a
          2a00:1450:400b:c02::65
          2a00:1450:400b:c02::66
          172.253.116.100
          172.253.116.138
          172.253.116.101
          172.253.116.113
          172.253.116.102
          172.253.116.139
```

## What I learned

The DNS server translated the domain name into an IP address.

The browser does not know where Google is located until DNS provides its IP address.

---

# Lab 2 — Inspect DNS Cache

## Objective

Verify that Windows stores DNS responses locally.

## Command

```powershell
ipconfig /displaydns
```

## Result

```text
Record Name . . . . . . . . . . . . . . : consent.config.office.com
Record Type . . . . . . . . . . . . . . : 5
Time To Live . . . . . . . . . . . . . : 7
Data Length . . . . . . . . . . . . . . : 8
Section . . . . . . . . . . . . . . . . : Answer
CNAME Record . . . . . . . . . . . . . : geo.consent.config.office.akadns.net
```



## What I learned

Windows caches DNS records to avoid querying the DNS server every time the same domain is accessed.

---

# Lab 3 — Flush DNS Cache

## Objective

Verify that the DNS cache can be cleared manually.

## Command

```powershell
ipconfig /flushdns
```

## Result

Successfully flushed the DNS Resolver Cache.

## What I learned

After clearing the cache, Windows needs to perform a new DNS lookup.

---

# Lab 4 — Compare Different DNS Servers

## Objective

Compare responses from different DNS resolvers.

## Commands

```powershell
nslookup github.com 8.8.8.8
```

```powershell
nslookup github.com 1.1.1.1
```

## Result

```text
Non-authoritative answer:
Name:    github.com
Address: 4.208.26.197
```


## What I learned

Different DNS resolvers may return different IP addresses depending on load balancing, caching or geographic location.

---

# Lab 5 — DNS vs IP

## Objective

Understand the difference between resolving a name and contacting an IP directly.

## Commands

```powershell
ping github.com
```

```powershell
ping 140.82.121.3
```

## What I learned

When using a domain name, Windows first performs DNS resolution.

When using an IP address, DNS is not required.
