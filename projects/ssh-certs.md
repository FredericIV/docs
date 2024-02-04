---
title: SSH Certificates
description: 
published: true
date: 2024-02-04T18:21:59.143Z
tags: 
editor: markdown
dateCreated: 2024-01-15T21:06:14.952Z
---

> WIP PAGE
{.is-warning}

# Certificates
## Status
- [X] CA stood up
- [X] Add CA to ca store
- [ ] Create host certificates
- [ ] Configure servers' sshd to accept ssh certificates

## Details
### CA stood up
### Add CA to system trust

```bash
#!/bin/bash
export ROOT_ADDR="https://ca.fabiv.pw/roots.pem" # Must be pem
export CERT_NAME="ca.fabiv.pw" # The right extension is auto-added
if [ "$EUID" -eq 0 ]
then
    export ESCALATE=""
elif command -v sudo &> /dev/null
    export ESCALATE="sudo"
elif command -v doas &> /dev/null
    export ESCALATE="doas"
else
    echo "What is going on here??? Neither sudo nor doas found."
    exit 1
fi
if [[ $(command -v curl &> /dev/null) -ne 0 ]]
then
  echo "Missing curl"
  exit 1
fi

case $(. /etc/os-release && echo $ID) in
  fedora)
    ${ESCALATE} curl -sko /etc/pki/ca-trust/source/anchors/${CERT_NAME}.pem ${ROOT_ADDR}
    ${ESCALATE} update-ca-trust
  alpine)
    if [[ $(command -v openssl &> /dev/null) -ne 0 ]]
    then
      echo "missing openssl"
      exit 1
    fi
    pushd /tmp
    curl -ko root.pem ${ROOT_ADDR}
    openssl x509 -inform PEM -in root.pem -out ${CERT_NAME}.crt
    rm root.pem
    ${ESCALATE} mv ${CERT_NAME}.crt /usr/local/share/ca-certificates/
    ${ESCALATE} update-ca-certificates
  debian)
    if [[ $(command -v openssl &> /dev/null) -ne 0 ]]
    then
      echo "missing openssl"
      exit 1
    fi
    pushd /tmp
    curl -ko root.pem ${ROOT_ADDR}
    openssl x509 -inform PEM -in root.pem -out ${CERT_NAME}.crt
    rm root.pem
    ${ESCALATE} mv ${CERT_NAME}.crt /usr/local/share/ca-certificates/
    ${ESCALATE} dpkg-reconfigure ca-certificates
    
```