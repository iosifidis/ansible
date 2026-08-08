---
layout: default
title: "Κεφάλαιο 4: Εγκατάσταση Ansible"
sort: 4
---

# Κεφάλαιο 4: Εγκατάσταση Ansible 📦

## Τι θα καλύψουμε

Στο Κεφάλαιο 3 εγκαταστήσαμε την Ansible στο **Armbian (Debian-based)**. Τώρα θα το κάνουμε **σωστά και για τις 3 διανομές**, και θα εξηγήσουμε **γιατί** κάθε βήμα.

---

## Δομή

```
4.1 Προαπαιτούμενα
    ├── Τι χρειάζεται η Ansible για να τρέξει
    ├── Python: έλεγχος & εγκατάσταση
    │   ├── Debian/Ubuntu
    │   ├── RedHat/Fedora
    │   └── openSUSE
    └── pip & venv: έλεγχος & εγκατάσταση
        ├── Debian/Ubuntu
        ├── RedHat/Fedora
        └── openSUSE

4.2 Μέθοδοι Εγκατάστασης
    ├── Μέθοδος 1: OS package manager
    │   ├── apt (Debian/Ubuntu)
    │   ├── dnf (RedHat/Fedora)
    │   └── zypper (openSUSE)
    ├── Μέθοδος 2: pip + venv ← συνιστάται
    └── Μέθοδος 3: pipx
        └── Σύγκριση και των 3

4.3 Εγκατάσταση με pip + venv
    ├── Debian/Ubuntu (αναλυτικά)
    ├── RedHat/Fedora (αναλυτικά) ← πρακτικό!
    └── openSUSE (αναλυτικά)     ← πρακτικό!

4.4 Επαλήθευση Εγκατάστασης
    ├── ansible --version (και τι σημαίνει κάθε γραμμή)
    ├── ansible --version (αναμενόμενο output ανά distro)
    └── Πρώτο ping test

4.5 Εγκατάσταση Collections
    ├── Τι είναι τα collections (σύντομη υπενθύμιση)
    ├── community.general
    ├── community.docker
    ├── ansible.posix
    └── Επαλήθευση

4.6 Αυτοματοποίηση με Playbook
    └── install-ansible.yml
        ├── Δουλεύει και στις 3 distros
        ├── Εγκαθιστά Python, venv, Ansible
        └── Εγκαθιστά Collections
```

## Υποπαράγραφοι
- [4.1 Προαπαιτούμενα](./4.1-prerequisites.md)   
- [4.2 Μέθοδοι εγκατάστασης Ansible](./4.2-installation-methods.md)   
- [4.3 Εγκατάσταση Ansible ανά διανομή](./4.3-installation-per-distro.md)   
- [4.4 Επαλήθευση εγκατάστασης](./4.4-verification.md)   
- [4.5 Εγκατάσταση Collections](./4.5-collections-installation.md)   
- [4.6 Αυτοματοποίηση Εγκατάστασης με Playbook](./4.6-automation-with-playbooks.md)   

> 💡 **Σημαντικό:** Αυτό το [`install-ansible-controller.yml`](../../files/ansible/install-ansible-controller.yml) είναι ήδη **παραγωγικό playbook** — είναι το **"bootstrap"** του controller σου. Το τρέχεις μία φορά σε ένα φρέσκο σύστημα και έχεις έτοιμο Ansible controller!
