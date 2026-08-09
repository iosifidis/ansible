---
layout: default
title: "Κεφάλαιο 17: Error Handling & Debugging"
sort: 17
---

# Κεφάλαιο 17: Error Handling & Debugging 🛡️

## Τι θα καλύψουμε

```
17.1 Τι είναι το Error Handling
     ├── Γιατί χρειάζεται
     └── Πώς αντιδρά η Ansible σε σφάλματα

17.2 ignore_errors & failed_when
     ├── Πότε να αγνοούμε σφάλματα
     └── Ορισμός δικού μας κριτηρίου αποτυχίας

17.3 changed_when
     └── Έλεγχος πότε ένα task «άλλαξε» κάτι

17.4 Block / Rescue / Always
     ├── Η try-catch της Ansible
     ├── rescue: — τι γίνεται αν αποτύχει
     └── always: — εκτελείται πάντα

17.5 Debug Module & Verbose Mode
     ├── debug module
     ├── assert module
     └── -v, -vv, -vvv, -vvvv

17.6 Πρακτικό παράδειγμα
     └── Full playbook με error handling
```

## Υποπαράγραφοι
- [17.1 Τι είναι το Error Handling 🛡️](./17.1-what-is-error-handling.md)
- [17.2 ignore_errors & failed_when ❌](./17.2-ignore-errors-failed-when.md)
- [17.3 changed_when 🔄](./17.3-changed-when.md)
- [17.4 Block / Rescue / Always 🏗️](./17.4-block-rescue-always.md)
- [17.5 Debug Module & Verbose Mode 🔍](./17.5-debug-verbose.md)
- [17.6 Πρακτικό παράδειγμα Error Handling 🛠️](./17.6-error-handling-example.md)
