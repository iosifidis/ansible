---
layout: default
title: "Full Stack Deployment"
sort: 6
---

# D6: Full Stack Deployment 🚀

## Τι θα φτιάξουμε

```
┌─────────────────────────────────────────────────────────┐
│  Production Full Stack                                  │
│                                                         │
│  Internet :80/:443                                      │
│       │                                                 │
│       ▼                                                 │
│  ┌─────────┐                                            │
│  │  Nginx  │ ← Reverse proxy + SSL                      │
│  └────┬────┘                                            │
│       │                                                 │
│  ┌────┴─────────────────┐                               │
│  ▼                      ▼                               │
│  ┌─────────┐    ┌──────────────┐                        │
│  │  MyApp  │    │  Adminer     │ ← DB management        │
│  │ (node)  │    │  (web UI)    │                        │
│  └────┬────┘    └──────────────┘                        │
│       │                                                 │
│  ┌────┴──────────────┐                                  │
│  ▼                   ▼                                  │
│  ┌──────────┐  ┌──────────┐                             │
│  │ Postgres │  │  Redis   │                             │
│  └──────────┘  └──────────┘                             │
│                                                         │
│  Monitoring:                                            │
│  ┌──────────┐  ┌──────────┐                             │
│  │ cAdvisor │  │ Portainer│ ← Container management      │
│  └──────────┘  └──────────┘                             │
└─────────────────────────────────────────────────────────┘
```


## Υποπαράγραφοι
- [6.1 Full Stack Deployment](./6-complete-stack-deploy.md)

