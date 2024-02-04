---
title: Certificates
description: 
published: true
date: 2024-02-04T19:43:47.240Z
tags: 
editor: markdown
dateCreated: 2024-01-15T21:06:14.952Z
---

> WIP PAGE
{.is-warning}

# Status
- [X] CA stood up
- [X] Add CA to system trust store
- [ ] Create host certificates
- [ ] Configure servers' sshd to accept ssh certificates

# Details
## CA stood up
Modern step-ca uses a database for managing provisioners, requiring use of their cli. Instructions for adding OIDC and SSHPOP provisioners will be added at some point.

/etc/step-ca/config/ca.json
```json
{
	"root": "/etc/step-ca/certs/root_ca.crt",
	"crt": "/etc/step-ca/certs/intermediate_ca.crt",
	"key": "pkcs11:id=06",
	"federatedRoots": [],
	"kms": {
		"type": "pkcs11",
		"uri": "pkcs11:module-path=/usr/lib/x86_64-linux-gnu/libykcs11.so;slot-id=0?pin-value=XXXXXX"
	},
	"address": ":443",
	"insecureAddress": "",
	"dnsNames": [
		"ca.fabiv.pw"
	],
	"ssh": {
 		"hostKey": "pkcs11:id=07",
		"userKey": "pkcs11:id=08"
	},
	"logger": {
		"format": "text"
	},
	"db": {
		"type": "badgerv2",
		"dataSource": "/etc/step-ca/db",
		"badgerFileLoadingMode": ""
	},
	"authority": {
		"enableAdmin": true
	},
	"tls": {
		"cipherSuites": [
			"TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256",
			"TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256"
		],
		"minVersion": 1.2,
		"maxVersion": 1.3,
		"renegotiation": false
	}
}
```
/etc/step-ca/config/defaults.json
```json
{
	"ca-url": "https://ca.fabiv.pw",
	"ca-config": "/etc/step-ca/config/ca.json",
	"fingerprint": "12452d6e3f40376ef775c8a0a821b8b91205d07c5249995f04da610c2c0e2e2b",
	"root": "/etc/step-ca/certs/root_ca.crt"
}
```
/etc/systemd/system/step-ca.service
```ini
[Unit]
Description=step-ca service
Documentation=https://smallstep.com/docs/step-ca
Documentation=https://smallstep.com/docs/step-ca/certificate-authority-server-production
After=network-online.target
Wants=network-online.target
ConditionFileNotEmpty=/etc/step-ca/config/ca.json
After=smartcard.target

[Service]
Type=simple
User=step
Group=step
Environment=STEPPATH=/etc/step-ca
WorkingDirectory=/etc/step-ca
ExecStart=/usr/local/bin/step-ca config/ca.json
ExecReload=/bin/kill --signal HUP $MAINPID
Restart=on-failure
RestartSec=5
TimeoutStopSec=30
StartLimitInterval=30
StartLimitBurst=3

; Process capabilities & privileges
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
SecureBits=keep-caps
NoNewPrivileges=yes

; Sandboxing
; This sandboxing works with YubiKey PIV (via pcscd HTTP API), but it is likely
; too restrictive for PKCS#11 HSMs.
;
; NOTE: Comment out the rest of this section for troubleshooting.
ProtectSystem=full
ProtectHome=true
RestrictNamespaces=true
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
PrivateTmp=true
ProtectClock=true
ProtectControlGroups=true
ProtectKernelTunables=true
ProtectKernelLogs=true
ProtectKernelModules=true
LockPersonality=true
RestrictSUIDSGID=true
RemoveIPC=true
RestrictRealtime=true
;PrivateDevices=true ; We need to access the yubikey from the host
SystemCallFilter=@system-service
SystemCallArchitectures=native
MemoryDenyWriteExecute=true
ReadWriteDirectories=/etc/step-ca/db


[Install]
WantedBy=smartcard.target

```
## Add CA to system trust store
> Make sure the host can reach the server with the certificate (generally the CA)!
{.is-warning}

```bash
#!/bin/bash
set -x
export ROOT_ADDR="https://ca.fabiv.pw/roots.pem" # Must be pem
export CERT_NAME="ca.fabiv.pw" # The right extension is auto-added
if [ "$EUID" -eq 0 ]
then
    export ESCALATE=""
elif command -v sudo &> /dev/null
then
    export ESCALATE="sudo"
elif command -v doas &> /dev/null
then
    export ESCALATE="doas"
else
    echo "What is going on here??? Neither sudo nor doas found."
    exit 1
fi
if ! command -v curl &> /dev/null
then
  echo "Missing curl"
  exit 1
fi
export ID=$(. /etc/os-release && echo $ID)
if [[ $(. /etc/os-release && echo $ID_LIKE) ]]; then export ID=$(. /etc/os-release && echo $ID_LIKE); fi
case ${ID} in
  fedora)
    export PKIDIR="/etc/pki/ca-trust/source/anchors"
    if [ ! -d ${PKIDIR} ]; then
      mkdir -p ${PKIDIR}
    fi
    ${ESCALATE} curl -sko ${PKIDIR}/${CERT_NAME}.pem ${ROOT_ADDR}
    ${ESCALATE} update-ca-trust
    ;;

  alpine)
    if ! command -v openssl &> /dev/null
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
    popd
    ;;

  debian)
    if ! command -v openssl &> /dev/null
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
    popd
    ;;
esac
```