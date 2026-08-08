---
layout: default
title: Ansible Files
sort: 3
---

# 📁 Ansible Files & Downloadable Resources

Καλώς ήρθατε στη βιβλιοθήκη αρχείων! Τα αρχεία αυτής της σελίδας **ενημερώνονται αυτόματα** κάθε φορά που προσθέτετε νέα αρχεία στους φακέλους `files/ansible/` και `files/docker/`.

---

## 🛠️ Ansible Playbooks & Configs (`files/ansible/`)

{% assign ansible_files = site.static_files | where_exp: "item", "item.path contains '/files/ansible/'" %}

{% if ansible_files.size > 0 %}
{% for file in ansible_files %}
- [📥 **{{ file.name }}**]({{ file.path | relative_url }})
{% endfor %}
{% else %}
*Δεν υπάρχουν ακόμα διαθέσιμα αρχεία στον φάκελο `files/ansible/`.*
{% endif %}

---

## 🐳 Docker Automation Files (`files/docker/`)

{% assign docker_files = site.static_files | where_exp: "item", "item.path contains '/files/docker/'" %}

{% if docker_files.size > 0 %}
{% for file in docker_files %}
- [📥 **{{ file.name }}**]({{ file.path | relative_url }})
{% endfor %}
{% else %}
*Δεν υπάρχουν ακόμα διαθέσιμα αρχεία στον φάκελο `files/docker/`.*
{% endif %}

---

## 💡 Πώς λειτουργεί η αυτόματη προσθήκη
Απλώς τοποθετήστε οποιοδήποτε αρχείο (π.χ. `.yml`, `.sh`, `.zip`, `.pdf`, `.png`) στον φάκελο `files/ansible/` ή `files/docker/`.
Το σύστημα εντοπίζει **αυτόματα** όλα τα νέα αρχεία και δημιουργεί τους συνδέσμους λήψης χωρίς να χρειάζεται να κάνετε καμία χειροκίνητη αλλαγή στη σελίδα!
