---
layout: default
title: Αρχική - Ansible & Docker Tutorial
sort: 0
---

# 🚀 Εγχειρίδια Αυτοματισμού Ansible & Docker

Καλώς ήρθατε στην κεντρική πύλη εκπαιδευτικού υλικού! Εδώ θα βρείτε πλήρεις οδηγούς για το **Ansible (16 Κεφάλαια)**, το **Docker Automation (6 Κεφάλαια)** καθώς και συνοδευτικά αρχεία λήψης (**Ansible Files**).

---

## 📖 Πίνακας Περιεχομένων & Κατάσταση Πρόοδου

Ο παρακάτω πίνακας ενημερώνεται **αυτόματα** από το σύστημα κάθε φορά που προσθέτετε ή τροποποιείτε κεφάλαια!

### 🎯 1. Ansible Learning Path (16 Κεφάλαια)

{% assign ansible_pages = site.html_pages | where_exp: "item", "item.path contains 'ansible/'" | sort: "path" %}

| Τίτλος Ενότητας / Υποπαραγράφου | Κατάσταση | Σύνδεσμος |
| :--- | :---: | :--- |
{% for p in ansible_pages %}{% if p.url != '/ansible/' %} | **{{ p.title }}** | {% if p.completed == true %}✅ Ολοκληρώθηκε{% else %}📝 Πρότυπο{% endif %} | [Προβολή →]({{ p.url | relative_url }}) |
{% endif %}{% endfor %}

---

### 🐳 2. Docker Automation με Ansible (6 Κεφάλαια)

{% assign docker_pages = site.html_pages | where_exp: "item", "item.path contains 'docker/'" | sort: "path" %}

| Τίτλος Ενότητας / Υποπαραγράφου | Κατάσταση | Σύνδεσμος |
| :--- | :---: | :--- |
{% for p in docker_pages %}{% if p.url != '/docker/' %} | **{{ p.title }}** | {% if p.completed == true %}✅ Ολοκληρώθηκε{% else %}📝 Πρότυπο{% endif %} | [Προβολή →]({{ p.url | relative_url }}) |
{% endif %}{% endfor %}

---

## 📁 3. Αρχεία Λήψης

Όλα τα συνοδευτικά αρχεία (Playbooks, Docker Compose κλπ.) είναι διαθέσιμα στην ενότητα [Ansible Files]({{ '/files/' | relative_url }}).

---
> 💡 *Για οδηγίες σχετικά με τη συγγραφή περιεχομένου, ανατρέξτε στο αρχείο [Instructions.md]({{ '/Instructions.html' | relative_url }}).*
