---
title: Projecs
description: 
published: true
date: 2024-01-15T20:47:45.197Z
tags: docker
editor: markdown
dateCreated: 2024-01-15T20:47:45.197Z
---

# Projects
## In Progres
### SSH Certificates
- [X] CA stood up
- [ ] Distribute host certificates
- [ ] Configure servers' sshd to accept ssh certificates
### HA DHCP & DNS
- [X] Kea conf generated
- [ ] PDNS-Auth conf generated.
  - [ ] SQL generated
  - [ ] Service conf generated
- [ ] PDNS-Resolver conf generated
- [ ] HA Postgres
  - [X] Conf Generated
  - [ ] Failover Tested
> Because PDNS-Auth is using the postgres backend, DNS configuration needs to happen either in the CLI or using SQL directly.
{.is-info}
## In Consideration
- Router OS to replace Vyos
  - VPP & FRR based
  - HA Capable
  - Built in DHCP & DNS
  - Firewall vs ACL f/ VPP?
  - Declaritive config syntax
## Complete
- Graphviz previewer
- K8s management container