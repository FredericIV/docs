---
title: Snippets
description: 
published: true
date: 2023-09-03T22:53:35.856Z
tags: k8s, cli
editor: markdown
dateCreated: 2023-09-03T22:52:55.299Z
---

Remove unwanted namespaces
```bash
NAMESPACE=your-rogue-namespace
kubectl proxy &
kubectl get namespace $NAMESPACE -o json |jq '.spec = {"finalizers":[]}' >temp.json
curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8001/api/v1/namespaces/$NAMESPACE/finalize
fg #ctrl+c
```