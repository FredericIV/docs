---
title: Projects
description: 
published: true
date: 2024-02-04T19:43:20.004Z
tags: 
editor: markdown
dateCreated: 2024-01-15T20:47:45.197Z
---

# Projects
## In Progres
- [HA DHCP & DNS](/projects/ha-dhcp-dns)
- [Certificates](/projects/certs)
- [Vyos Config Generation](/projects/vyos)
  - GA depends on HA DHCP & DNS
## In Consideration
- Router OS to replace Vyos
  - VPP & FRR based
  - HA Capable
  - Built in DHCP & DNS (deps on [HA DHCP & DNS](/projects/ha-dhcp-dns))
  - Firewall vs ACL f/ VPP?
  - Declaritive config syntax
- Implement CI/CD (Concourse CI)
  - Auto-build docker containers
  - Migrate from Wiki.js to mkdocs-material
  - deps on DNS integration
- Bare-metal openstack cluster
- Migrate to Dex as OIDC intermediary for k8s
  - deps on DNS integration
- Integrate k8s DNS & int DNS
  - [External-DNS](https://github.com/kubernetes-sigs/external-dns/tree/master)
  - deps on [HA DHCP & DNS](/projects/ha-dhcp-dns)
## Complete
- Graphviz previewer
- K8s management container