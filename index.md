---
layout: default
title: Αρχική - Ansible & Docker Tutorial
sort: 0
---

# 🚀 Εγχειρίδια Αυτοματισμού Ansible & Docker

Καλώς ήρθατε στην κεντρική πύλη εκπαιδευτικού υλικού! Εδώ θα βρείτε τους πλήρεις οδηγούς για το **Ansible Learning Path (16 Κεφάλαια)**, το **Docker Automation (6 Κεφάλαια)** καθώς και συνοδευτικά αρχεία λήψης (**Ansible Files**).

---

## 📖 Πίνακας Περιεχομένων

### 🎯 1. Ansible Learning Path (16 Κεφάλαια)

{% assign ansible_chapters = site.html_pages | where_exp: "item", "item.path contains 'ansible/chapter-'" | where_exp: "item", "item.name == 'index.md'" | sort: "path" %}

| Κεφάλαιο | Σύνδεσμος |
| :--- | :--- |
{% for p in ansible_chapters %} | **{{ p.title }}** | [Προβολή →]({{ p.url | relative_url }}) |
{% endfor %}

---

### 🐳 2. Docker Automation με Ansible (6 Κεφάλαια)

{% assign docker_chapters = site.html_pages | where_exp: "item", "item.path contains 'docker/chapter-'" | where_exp: "item", "item.name == 'index.md'" | sort: "path" %}

| Κεφάλαιο | Σύνδεσμος |
| :--- | :--- |
{% for p in docker_chapters %} | **{{ p.title }}** | [Προβολή →]({{ p.url | relative_url }}) |
{% endfor %}

---

## 📁 3. Αρχεία Λήψης

Όλα τα συνοδευτικά αρχεία (Playbooks, Docker Compose κλπ.) είναι διαθέσιμα στην ενότητα [Ansible Files]({{ '/files/' | relative_url }}).

---
> 💡 *Για οδηγίες σχετικά με τη συγγραφή περιεχομένου, ανατρέξτε στο αρχείο [Instructions.md]({{ '/Instructions.html' | relative_url }}).*
