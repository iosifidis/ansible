---
layout: default
title: "Κεφάλαιο 18: Performance & Parallel Execution"
sort: 18
---

# Κεφάλαιο 18: Performance & Parallel Execution ⚡

## Τι θα καλύψουμε

```
18.1 Πώς εκτελεί η Ansible tasks
     ├── Linear execution (default)
     └── Πόσο γρήγορη είναι;

18.2 forks — Παράλληλη εκτέλεση
     ├── Τι είναι τα forks
     ├── Ρύθμιση στο ansible.cfg
     └── Ρύθμιση ανά εκτέλεση

18.3 serial — Rolling Updates
     ├── Γιατί χρειάζεται το serial
     ├── Αριθμητικό & ποσοστιαίο serial
     └── max_fail_percentage

18.4 async & poll — Long-running Tasks
     ├── Πότε χρειάζεται
     ├── async: — timeout
     ├── poll: — έλεγχος κατάστασης
     └── Fire-and-forget pattern

18.5 Strategy — Στρατηγική εκτέλεσης
     ├── linear (default)
     ├── free
     └── host_pinned

18.6 Πρακτικό παράδειγμα
     └── Rolling update με zero downtime
```

## Υποπαράγραφοι
- [18.1 Πώς εκτελεί η Ansible tasks ⚡](./18.1-how-ansible-executes.md)
- [18.2 forks — Παράλληλη εκτέλεση 🔀](./18.2-forks.md)
- [18.3 serial — Rolling Updates 🔄](./18.3-serial-rolling-updates.md)
- [18.4 async & poll — Long-running Tasks ⏳](./18.4-async-poll.md)
- [18.5 Strategy — Στρατηγική εκτέλεσης 🎯](./18.5-strategy.md)
- [18.6 Πρακτικό παράδειγμα — Rolling Update με Zero Downtime 🚀](./18.6-rolling-update-example.md)
