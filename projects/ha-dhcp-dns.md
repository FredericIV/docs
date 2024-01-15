---
title: HA DHCP & DNS
description: 
published: true
date: 2024-01-15T21:03:30.128Z
tags: docker
editor: markdown
dateCreated: 2024-01-15T21:03:30.128Z
---

# HA DHCP & DNS
## Status
- [X] Kea conf generated
- [ ] PDNS-Auth conf generated.
  - [ ] SQL generated
  - [ ] Service conf generated
- [ ] PDNS-Resolver conf generated
- [ ] HA Postgres
  - [X] Conf Generated
  - [ ] Failover Tested
## Rational
### Kea
### DNS
A [frequently cited guideline](https://kb.isc.org/docs/bind-best-practices-authoritative#2-run-separate-authoritative-and-recursive-dns-servers) for DNS states that recursive DNS and authoritative DNS servers should remain seperate. To this end, DNS servers may generaly be found in three camps: those with seperate functions and modern capabilities/sensabilites, and those that are aimed at SOHO applications, and those that are just old. Bind falls into the last camp, and dnsmasq falls into the last two camps. PowerDNS was chosen for its modern and easy nature, with the understanding that a platform that can afford to use containers can probably afford to run both an authoritative and a recursive DNS server. The database backend is also much easier to work with. 
#### PDNS-Auth
Because PDNS-Auth is using the postgres backend, DNS configuration needs to happen either in the CLI or using SQL directly. To avoid any extra dependencies and because it's the first thing I thought of, SQL is utilized.
#### PDNS-Recursive
### Postgres