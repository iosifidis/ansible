---
layout: default
title: Κεφάλαιο 3: Environment Setup
sort: 3
---

# Κεφάλαιο 3: Environment Setup 🛠️

```
✅ Armbian → Ansible Controller
✅ ansible user με SSH key (ed25519)
✅ Virtual environment με Ansible 2.20.4
✅ Σωστή δομή φακέλων εργασίας
✅ Managed server (nextcloudpi) έτοιμος
✅ Πρώτο real-world Security Playbook
   ├── SSH port → 2022
   ├── Root login → απενεργοποιημένο
   ├── Password auth → απενεργοποιημένη
   ├── ferm firewall → ενεργό
   └── fail2ban → ενεργό
```

## Τι θα κάνουμε σε αυτό το κεφάλαιο

Θα στήσουμε ένα **πραγματικό περιβάλλον** με το δικό σου hardware:

```
Τι θα στήσουμε:

┌─────────────────────────────────────────────────┐
│  Ubuntu PC σου                                  │
│  └── SSH ──► Armbian (Ansible Controller)       │
│              ├── ans_ctrl user                  │
│              ├── SSH key (ed25519)              │
│              ├── Ansible εγκατεστημένο          │
│              └── SSH ──► Managed Servers        │
└─────────────────────────────────────────────────┘
```

## Τα βήματα μας:
- [3.1 Προετοιμασία Armbian 🖥️](./3.1-preparation.md)   
- [3.2 Δημιουργία `ansible` User 👤](./3.2-ansible-user.md)   
- [3.3 Δημιουργία SSH Key 🔑](./3.3-ssh-key.md)   
- [3.4 Ρύθμιση φακέλων εργασίας 📁](./3.4-folders.md)   
- [3.5 Εγκατάσταση Ansible 📦](./3.5-ansible-install.md)   
- [3.6 Προετοιμασία Managed Servers 🖥️](./3.6-managed-servers-preparation.md)   
- [3.7 Αποστολή SSH Key στον Managed Server 🔑](./3.7-key-on-managed-server.md)   
- [3.8 Πρώτο test σύνδεσης με Ansible 🚀](./3.8-connection-test.md)   
- [3.9 Security Playbook 🔐](./3.9-security-playbook.md)

> 💡 **Σημαντικό:** Αυτό το [`security-bootstrap.yml`](../../files/ansible/security-bootstrap.yml) είναι ήδη **παραγωγικό εργαλείο** — μπορείς να το τρέξεις σε κάθε νέο server που προσθέτεις!
