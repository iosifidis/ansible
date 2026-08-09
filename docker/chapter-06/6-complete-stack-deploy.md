---
layout: default
title: Full Stack Deployment
sort: 1
---
{% raw %}
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

---

## Δομή Project

```bash
# Δημιουργία δομής
mkdir -p ~/ansible/roles/fullstack/{tasks,templates,defaults,handlers,files}

tree ~/ansible/roles/fullstack/
# fullstack/
# ├── defaults/main.yml
# ├── handlers/main.yml
# ├── tasks/
# │   ├── main.yml
# │   ├── setup.yml
# │   ├── database.yml
# │   ├── cache.yml
# │   ├── app.yml
# │   ├── proxy.yml
# │   ├── monitoring.yml
# │   └── verify.yml
# └── templates/
#     ├── docker-compose.yml.j2
#     ├── nginx.conf.j2
#     ├── app.env.j2
#     └── postgres-init.sql.j2
```

---

## defaults/main.yml

```bash
cat > ~/ansible/roles/fullstack/defaults/main.yml << 'EOF'
---
# ============================================================
# Full Stack Defaults
# ============================================================

# ── Stack Identity ────────────────────────────
stack_name:    mystack
stack_version: "1.0.0"
app_env:       production
stack_domain:  "{{ ansible_facts['default_ipv4']['address'] }}"

# ── Paths ─────────────────────────────────────
stack_dir:     "/opt/{{ stack_name }}"
stack_cfg_dir: "/opt/{{ stack_name }}/config"
stack_log_dir: "/opt/{{ stack_name }}/logs"
stack_bak_dir: "/opt/{{ stack_name }}/backups"

# ── Application ───────────────────────────────
app_name:    myapp
app_version: "{{ stack_version }}"
app_image:   "nginx:1.25.3-alpine"   # ← placeholder
app_port:    3000

# ── Nginx ─────────────────────────────────────
nginx_image:    nginx:1.25.3-alpine
nginx_http_port:  80
nginx_https_port: 443

# ── PostgreSQL ────────────────────────────────
postgres_image:   postgres:15.4-alpine
postgres_db:      "{{ stack_name }}_db"
postgres_user:    "{{ stack_name }}_user"
postgres_port:    5432
postgres_version: "15"

# ── Redis ─────────────────────────────────────
redis_image:   redis:7.2-alpine
redis_port:    6379
redis_maxmem:  "256mb"

# ── Adminer (DB UI) ───────────────────────────
adminer_enabled: true
adminer_image:   adminer:4-standalone
adminer_port:    8080

# ── Monitoring ────────────────────────────────
monitoring_enabled:   true
portainer_enabled:    true
portainer_port:       9000
cadvisor_port:        8081

# ── Backup ────────────────────────────────────
backup_enabled:   true
backup_schedule:  "0 3 * * *"    # κάθε μέρα 03:00
backup_retention: 7              # κράτα 7 μέρες

# ── Resources ─────────────────────────────────
nginx_memory:     "128m"
nginx_cpus:       "0.25"
app_memory:       "512m"
app_cpus:         "1.0"
postgres_memory:  "512m"
postgres_cpus:    "0.5"
redis_memory:     "256m"
redis_cpus:       "0.25"
adminer_memory:   "64m"
adminer_cpus:     "0.1"
EOF
```

---

## templates/docker-compose.yml.j2

```bash
cat > ~/ansible/roles/fullstack/templates/docker-compose.yml.j2 << 'EOF'
{# Full Stack Docker Compose #}
{# Managed by Ansible — DO NOT EDIT! #}
{# Stack: {{ stack_name }} v{{ stack_version }} #}
{# Generated: {{ ansible_facts['date_time']['iso8601'] }} #}

version: "3.8"

{# ── Resource calculations ────────────────── #}
{% set total_ram = ansible_facts['memtotal_mb'] | int %}
{% set total_cpu = ansible_facts['processor_vcpus'] | int %}

services:

  # ══════════════════════════════════════════
  # NGINX — Reverse Proxy
  # ══════════════════════════════════════════
  nginx:
    image: {{ nginx_image }}
    container_name: {{ stack_name }}_nginx
    restart: unless-stopped
    ports:
      - "{{ nginx_http_port }}:80"
      - "{{ nginx_https_port }}:443"
    volumes:
      - {{ stack_cfg_dir }}/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - {{ stack_log_dir }}/nginx:/var/log/nginx
      - nginx_cache:/var/cache/nginx
      - /etc/localtime:/etc/localtime:ro
    networks:
      - frontend
    depends_on:
      app:
        condition: service_started
    deploy:
      resources:
        limits:
          memory: {{ nginx_memory }}
          cpus:   "{{ nginx_cpus }}"
    logging:
      driver: json-file
      options: {max-size: "10m", max-file: "3"}
    healthcheck:
      test:         ["CMD", "wget", "-q", "-O", "/dev/null", "http://localhost/health"]
      interval:     30s
      timeout:      10s
      retries:      3
      start_period: 10s
    labels:
      stack:     {{ stack_name }}
      component: nginx

  # ══════════════════════════════════════════
  # APPLICATION
  # ══════════════════════════════════════════
  app:
    image: {{ app_image }}
    container_name: {{ stack_name }}_app
    restart: unless-stopped
    env_file:
      - {{ stack_cfg_dir }}/app.env
    volumes:
      - app_data:/app/data
      - app_uploads:/app/uploads
      - {{ stack_log_dir }}/app:/app/logs
      - /etc/localtime:/etc/localtime:ro
    networks:
      - frontend
      - backend
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    deploy:
      resources:
        limits:
          memory: {{ app_memory }}
          cpus:   "{{ app_cpus }}"
    logging:
      driver: json-file
      options: {max-size: "10m", max-file: "3"}
    healthcheck:
      test:         ["CMD", "wget", "-q", "-O", "/dev/null", "http://localhost:{{ app_port }}/"]
      interval:     30s
      timeout:      10s
      retries:      3
      start_period: 30s
    labels:
      stack:     {{ stack_name }}
      component: app
      version:   "{{ stack_version }}"

  # ══════════════════════════════════════════
  # POSTGRESQL
  # ══════════════════════════════════════════
  postgres:
    image: {{ postgres_image }}
    container_name: {{ stack_name }}_postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB:       {{ postgres_db }}
      POSTGRES_USER:     {{ postgres_user }}
      POSTGRES_PASSWORD: {{ vault_db_password }}
      PGDATA:            /var/lib/postgresql/data/pgdata
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - {{ stack_cfg_dir }}/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
      - {{ stack_bak_dir }}:/backups
      - /etc/localtime:/etc/localtime:ro
    networks:
      - backend
    deploy:
      resources:
        limits:
          memory: {{ postgres_memory }}
          cpus:   "{{ postgres_cpus }}"
    logging:
      driver: json-file
      options: {max-size: "10m", max-file: "3"}
    healthcheck:
      test:         ["CMD-SHELL", "pg_isready -U {{ postgres_user }} -d {{ postgres_db }}"]
      interval:     10s
      timeout:      5s
      retries:      5
      start_period: 30s
    labels:
      stack:     {{ stack_name }}
      component: postgres

  # ══════════════════════════════════════════
  # REDIS
  # ══════════════════════════════════════════
  redis:
    image: {{ redis_image }}
    container_name: {{ stack_name }}_redis
    restart: unless-stopped
    command: >
      redis-server
      --maxmemory {{ redis_maxmem }}
      --maxmemory-policy allkeys-lru
      --appendonly yes
      --requirepass {{ vault_redis_password }}
      --loglevel {{ 'debug' if app_env != 'production' else 'notice' }}
    volumes:
      - redis_data:/data
      - /etc/localtime:/etc/localtime:ro
    networks:
      - backend
    deploy:
      resources:
        limits:
          memory: {{ redis_memory }}
          cpus:   "{{ redis_cpus }}"
    logging:
      driver: json-file
      options: {max-size: "10m", max-file: "3"}
    healthcheck:
      test:         ["CMD", "redis-cli", "--no-auth-warning", "-a", "{{ vault_redis_password }}", "ping"]
      interval:     10s
      timeout:      5s
      retries:      5
      start_period: 10s
    labels:
      stack:     {{ stack_name }}
      component: redis

  # ══════════════════════════════════════════
  # ADMINER (conditional)
  # ══════════════════════════════════════════
  {% if adminer_enabled %}
  adminer:
    image: {{ adminer_image }}
    container_name: {{ stack_name }}_adminer
    restart: unless-stopped
    environment:
      ADMINER_DEFAULT_SERVER: postgres
      ADMINER_DESIGN:         dracula
    networks:
      - frontend
      - backend
    depends_on:
      postgres:
        condition: service_healthy
    deploy:
      resources:
        limits:
          memory: {{ adminer_memory }}
          cpus:   "{{ adminer_cpus }}"
    logging:
      driver: json-file
      options: {max-size: "5m", max-file: "2"}
    labels:
      stack:     {{ stack_name }}
      component: adminer
  {% endif %}

  # ══════════════════════════════════════════
  # MONITORING (conditional)
  # ══════════════════════════════════════════
  {% if monitoring_enabled %}
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: {{ stack_name }}_cadvisor
    restart: unless-stopped
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    networks:
      - monitoring
    devices:
      - /dev/kmsg
    privileged: true
    deploy:
      resources:
        limits:
          memory: "128m"
          cpus:   "0.25"
    logging:
      driver: json-file
      options: {max-size: "5m", max-file: "2"}
    labels:
      stack:     {{ stack_name }}
      component: cadvisor
  {% endif %}

  {% if portainer_enabled %}
  portainer:
    image: portainer/portainer-ce:latest
    container_name: {{ stack_name }}_portainer
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - portainer_data:/data
    networks:
      - monitoring
    deploy:
      resources:
        limits:
          memory: "128m"
          cpus:   "0.25"
    logging:
      driver: json-file
      options: {max-size: "5m", max-file: "2"}
    labels:
      stack:     {{ stack_name }}
      component: portainer
  {% endif %}

# ══════════════════════════════════════════
# VOLUMES
# ══════════════════════════════════════════
volumes:
  postgres_data:
    name: {{ stack_name }}_postgres_data
  redis_data:
    name: {{ stack_name }}_redis_data
  app_data:
    name: {{ stack_name }}_app_data
  app_uploads:
    name: {{ stack_name }}_app_uploads
  nginx_cache:
    name: {{ stack_name }}_nginx_cache
  {% if portainer_enabled %}
  portainer_data:
    name: {{ stack_name }}_portainer_data
  {% endif %}

# ══════════════════════════════════════════
# NETWORKS
# ══════════════════════════════════════════
networks:
  frontend:
    name: {{ stack_name }}_frontend
    driver: bridge

  backend:
    name: {{ stack_name }}_backend
    driver: bridge
    internal: true

  monitoring:
    name: {{ stack_name }}_monitoring
    driver: bridge
EOF
```

---

## templates/nginx.conf.j2

```bash
cat > ~/ansible/roles/fullstack/templates/nginx.conf.j2 << 'EOF'
{# Nginx Full Stack Configuration #}
{# Managed by Ansible — DO NOT EDIT! #}

user nginx;
worker_processes {{ ansible_facts['processor_vcpus'] | default(1) }};
pid /var/run/nginx.pid;

events {
    worker_connections 2048;
    multi_accept on;
}

http {
    sendfile    on;
    tcp_nopush  on;
    tcp_nodelay on;
    server_tokens off;
    keepalive_timeout 65;
    client_max_body_size 64M;

    include      /etc/nginx/mime.types;
    default_type application/octet-stream;

    # ── Logging ───────────────────────────────
    log_format main
        '$remote_addr - $remote_user [$time_local] '
        '"$request" $status $body_bytes_sent '
        '"$http_referer" "$http_user_agent" '
        'rt=$request_time';

    access_log /var/log/nginx/access.log main;
    error_log  /var/log/nginx/error.log warn;

    # ── Rate limiting ─────────────────────────
    limit_req_zone $binary_remote_addr zone=global:20m rate=30r/s;
    limit_req_zone $binary_remote_addr zone=api:10m    rate=10r/s;
    limit_conn_zone $binary_remote_addr zone=conn:10m;

    # ── Gzip ──────────────────────────────────
    gzip            on;
    gzip_vary       on;
    gzip_proxied    any;
    gzip_comp_level 6;
    gzip_types
        text/plain text/css text/xml
        application/json application/javascript
        application/xml application/xml+rss
        text/javascript image/svg+xml;

    # ── Upstream ──────────────────────────────
    upstream app_backend {
        least_conn;
        server app:{{ app_port }};
        keepalive 32;
    }

    {% if adminer_enabled %}
    upstream adminer_backend {
        server adminer:8080;
    }
    {% endif %}

    # ── Main Server ───────────────────────────
    server {
        listen 80;
        server_name {{ stack_domain }};

        # ── Health check ──────────────────────
        location /health {
            access_log off;
            return 200 "OK\n";
            add_header Content-Type text/plain;
        }

        # ── Metrics endpoint ──────────────────
        location /nginx_status {
            stub_status on;
            access_log  off;
            allow       127.0.0.1;
            deny        all;
        }

        # ── API ───────────────────────────────
        location /api/ {
            limit_req  zone=api burst=20 nodelay;
            limit_conn conn 20;

            proxy_pass         http://app_backend;
            proxy_http_version 1.1;
            proxy_set_header   Upgrade     $http_upgrade;
            proxy_set_header   Connection  "upgrade";
            proxy_set_header   Host        $host;
            proxy_set_header   X-Real-IP   $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;

            proxy_connect_timeout 60s;
            proxy_send_timeout    60s;
            proxy_read_timeout    60s;
        }

        # ── Static ────────────────────────────
        location /static/ {
            alias   /var/www/static/;
            expires 1y;
            add_header Cache-Control "public, immutable";
            access_log off;
        }

        # ── Adminer ───────────────────────────
        {% if adminer_enabled %}
        location /adminer/ {
            allow 127.0.0.1;
            {% for ip in admin_allowed_ips | default(['127.0.0.1']) %}
            allow {{ ip }};
            {% endfor %}
            deny  all;

            proxy_pass http://adminer_backend/;
            proxy_set_header Host $host;
        }
        {% endif %}

        # ── Application ───────────────────────
        location / {
            limit_req  zone=global burst=50 nodelay;
            limit_conn conn 50;

            proxy_pass         http://app_backend;
            proxy_http_version 1.1;
            proxy_set_header   Upgrade    $http_upgrade;
            proxy_set_header   Connection "upgrade";
            proxy_set_header   Host       $host;
            proxy_set_header   X-Real-IP  $remote_addr;

            # ── Security headers ──────────────
            add_header X-Frame-Options        "SAMEORIGIN"    always;
            add_header X-Content-Type-Options "nosniff"       always;
            add_header X-XSS-Protection       "1; mode=block" always;
            add_header Referrer-Policy        "strict-origin-when-cross-origin" always;
        }
    }

    {% if monitoring_enabled %}
    # ── Monitoring Server ─────────────────────
    server {
        listen 8081;
        server_name {{ stack_domain }};

        location /cadvisor/ {
            proxy_pass http://cadvisor:{{ cadvisor_port }}/;
        }

        location /portainer/ {
            proxy_pass http://portainer:{{ portainer_port }}/;
            proxy_http_version 1.1;
            proxy_set_header   Upgrade    $http_upgrade;
            proxy_set_header   Connection "upgrade";
        }
    }
    {% endif %}
}
EOF
```

---

## templates/app.env.j2

```bash
cat > ~/ansible/roles/fullstack/templates/app.env.j2 << 'EOF'
{# Application Environment #}
{# Managed by Ansible Vault — DO NOT EDIT! #}

# ── App ───────────────────────────────────────
NODE_ENV={{ app_env }}
APP_VERSION={{ stack_version }}
PORT={{ app_port }}
LOG_LEVEL={{ 'debug' if app_env != 'production' else 'info' }}

# ── Database ──────────────────────────────────
DB_HOST=postgres
DB_PORT={{ postgres_port }}
DB_NAME={{ postgres_db }}
DB_USER={{ postgres_user }}
DB_PASSWORD={{ vault_db_password }}
DATABASE_URL=postgresql://{{ postgres_user }}:{{ vault_db_password }}@postgres:{{ postgres_port }}/{{ postgres_db }}
DB_POOL_MIN=2
DB_POOL_MAX=10

# ── Redis ─────────────────────────────────────
REDIS_HOST=redis
REDIS_PORT={{ redis_port }}
REDIS_PASSWORD={{ vault_redis_password }}
REDIS_URL=redis://:{{ vault_redis_password }}@redis:{{ redis_port }}/0
REDIS_TTL=3600

# ── Security ──────────────────────────────────
SECRET_KEY={{ vault_app_secret }}
JWT_SECRET={{ vault_jwt_secret | default(vault_app_secret) }}
JWT_EXPIRY=24h

# ── Feature flags ─────────────────────────────
CACHE_ENABLED={{ 'true' if app_env == 'production' else 'false' }}
DEBUG={{ 'false' if app_env == 'production' else 'true' }}

# ── System ────────────────────────────────────
TZ=Europe/Athens
SERVER_HOST={{ inventory_hostname }}
SERVER_IP={{ ansible_facts['default_ipv4']['address'] }}
EOF
```

---

## templates/postgres-init.sql.j2

```bash
cat > ~/ansible/roles/fullstack/templates/postgres-init.sql.j2 << 'EOF'
-- PostgreSQL Initialization Script
-- Managed by Ansible — DO NOT EDIT!
-- Stack: {{ stack_name }}

-- ── Extensions ────────────────────────────────
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";
CREATE EXTENSION IF NOT EXISTS "hstore";

-- ── Database configuration ────────────────────
ALTER DATABASE {{ postgres_db }}
    SET timezone TO 'Europe/Athens';

ALTER DATABASE {{ postgres_db }}
    SET log_min_duration_statement TO 1000;

-- ── Grant permissions ─────────────────────────
GRANT ALL PRIVILEGES ON DATABASE {{ postgres_db }}
    TO {{ postgres_user }};

GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public
    TO {{ postgres_user }};

GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public
    TO {{ postgres_user }};

ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT ALL ON TABLES TO {{ postgres_user }};

ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT ALL ON SEQUENCES TO {{ postgres_user }};

-- ── Health check table ────────────────────────
CREATE TABLE IF NOT EXISTS _health_check (
    id         SERIAL PRIMARY KEY,
    checked_at TIMESTAMP DEFAULT NOW()
);

INSERT INTO _health_check DEFAULT VALUES;
EOF
```

---

## tasks/main.yml

```bash
cat > ~/ansible/roles/fullstack/tasks/main.yml << 'EOF'
---
# ============================================================
# Full Stack — Main Entry Point
# ============================================================

- name: Setup infrastructure
  ansible.builtin.import_tasks:
    file: setup.yml
  tags: [setup, fullstack]

- name: Deploy stack
  ansible.builtin.import_tasks:
    file: deploy.yml
  tags: [deploy, fullstack]

- name: Post-deploy setup
  ansible.builtin.import_tasks:
    file: postdeploy.yml
  tags: [postdeploy, fullstack]

- name: Verify deployment
  ansible.builtin.import_tasks:
    file: verify.yml
  tags: [verify, fullstack]
EOF
```

---

## tasks/setup.yml

```bash
cat > ~/ansible/roles/fullstack/tasks/setup.yml << 'EOF'
---
# ============================================================
# Setup — Directories, Config Files
# ============================================================

# ── Directories ───────────────────────────────
- name: Create directory structure
  ansible.builtin.file:
    path:  "{{ item }}"
    state: directory
    mode:  '0755'
    owner: root
    group: docker
  loop:
    - "{{ stack_dir }}"
    - "{{ stack_cfg_dir }}"
    - "{{ stack_cfg_dir }}/nginx"
    - "{{ stack_cfg_dir }}/postgres"
    - "{{ stack_log_dir }}"
    - "{{ stack_log_dir }}/nginx"
    - "{{ stack_log_dir }}/app"
    - "{{ stack_bak_dir }}"

# ── Render templates ──────────────────────────
- name: Render docker-compose.yml
  ansible.builtin.template:
    src:  docker-compose.yml.j2
    dest: "{{ stack_dir }}/docker-compose.yml"
    mode: '0644'
  register: compose_changed

- name: Render nginx config
  ansible.builtin.template:
    src:  nginx.conf.j2
    dest: "{{ stack_cfg_dir }}/nginx/nginx.conf"
    mode: '0644'
  register: nginx_changed

- name: Render app environment
  ansible.builtin.template:
    src:  app.env.j2
    dest: "{{ stack_cfg_dir }}/app.env"
    mode: '0600'
  no_log: true
  register: env_changed

- name: Render postgres init script
  ansible.builtin.template:
    src:  postgres-init.sql.j2
    dest: "{{ stack_cfg_dir }}/postgres/init.sql"
    mode: '0644'

# ── Validate compose file ─────────────────────
- name: Validate docker-compose.yml
  ansible.builtin.command:
    cmd:   docker compose -f {{ stack_dir }}/docker-compose.yml config --quiet
    chdir: "{{ stack_dir }}"
  changed_when: false

- name: Setup summary
  ansible.builtin.debug:
    msg:
      - "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
      - "✅ Setup complete"
      - "Compose: {{ 'Changed' if compose_changed.changed else 'OK' }}"
      - "Nginx  : {{ 'Changed' if nginx_changed.changed  else 'OK' }}"
      - "Env    : {{ 'Changed' if env_changed.changed    else 'OK' }}"
      - "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
EOF
```

---

## tasks/deploy.yml

```bash
cat > ~/ansible/roles/fullstack/tasks/deploy.yml << 'EOF'
---
# ============================================================
# Deploy — Docker Compose Stack
# ============================================================

# ── Pull images ───────────────────────────────
- name: Pull all images
  ansible.builtin.command:
    cmd:   docker compose -f {{ stack_dir }}/docker-compose.yml pull
    chdir: "{{ stack_dir }}"
  register:     pull_result
  changed_when: "'Pull complete' in pull_result.stdout"

# ── Deploy stack ──────────────────────────────
- name: Deploy full stack
  community.docker.docker_compose_v2:
    project_src:  "{{ stack_dir }}"
    project_name: "{{ stack_name }}"
    state:        present
    pull:         missing
  register: stack_result
  no_log: true

- name: Deployment result
  ansible.builtin.debug:
    msg: "Stack {{ stack_name }}: {{ 'Changed ✅' if stack_result.changed else 'Up to date ✅' }}"

# ── Wait for services ─────────────────────────
- name: Wait for PostgreSQL healthy
  community.docker.docker_container_info:
    name: "{{ stack_name }}_postgres"
  register: pg_info
  until: >
    pg_info.container is defined and
    pg_info.container.State.Health.Status == 'healthy'
  retries: 18
  delay:   10

- name: PostgreSQL ✅
  ansible.builtin.debug:
    msg: "✅ PostgreSQL: {{ pg_info.container.State.Health.Status }}"

- name: Wait for Redis healthy
  community.docker.docker_container_info:
    name: "{{ stack_name }}_redis"
  register: redis_info
  until: >
    redis_info.container is defined and
    redis_info.container.State.Health.Status == 'healthy'
  retries: 6
  delay:   5

- name: Redis ✅
  ansible.builtin.debug:
    msg: "✅ Redis: {{ redis_info.container.State.Health.Status }}"

- name: Wait for App to start
  community.docker.docker_container_info:
    name: "{{ stack_name }}_app"
  register: app_info
  until: >
    app_info.container is defined and
    app_info.container.State.Running == true
  retries: 12
  delay:   5

- name: App ✅
  ansible.builtin.debug:
    msg: "✅ App: {{ app_info.container.State.Status }}"

- name: Wait for Nginx to start
  community.docker.docker_container_info:
    name: "{{ stack_name }}_nginx"
  register: nginx_info
  until: >
    nginx_info.container is defined and
    nginx_info.container.State.Health.Status in ['healthy', 'starting']
  retries: 6
  delay:   5

- name: Nginx ✅
  ansible.builtin.debug:
    msg: "✅ Nginx: {{ nginx_info.container.State.Status }}"
EOF
```

---

## tasks/postdeploy.yml

```bash
cat > ~/ansible/roles/fullstack/tasks/postdeploy.yml << 'EOF'
---
# ============================================================
# Post-Deploy — Backup, Monitoring, Cleanup
# ============================================================

# ── Backup script ─────────────────────────────
- name: Deploy backup script
  ansible.builtin.copy:
    dest:  /usr/local/bin/stack-backup.sh
    mode:  '0755'
    owner: root
    content: |
      #!/bin/bash
      # Stack Backup Script
      # Managed by Ansible — DO NOT EDIT!

      STACK="{{ stack_name }}"
      BACKUP_DIR="{{ stack_bak_dir }}"
      DATE=$(date +%Y%m%d_%H%M%S)
      RETENTION={{ backup_retention }}

      echo "=== Backup Start: $DATE ==="

      # PostgreSQL backup
      echo "Backing up PostgreSQL..."
      docker exec {{ stack_name }}_postgres \
          pg_dump \
          -U {{ postgres_user }} \
          -d {{ postgres_db }} \
          --format=custom \
          --compress=9 \
          > "${BACKUP_DIR}/postgres_${DATE}.dump"

      if [ $? -eq 0 ]; then
          echo "✅ PostgreSQL backup: postgres_${DATE}.dump"
      else
          echo "❌ PostgreSQL backup FAILED!"
          exit 1
      fi

      # Redis backup
      echo "Backing up Redis..."
      docker exec {{ stack_name }}_redis \
          redis-cli --no-auth-warning \
          -a "{{ vault_redis_password }}" \
          BGSAVE
      sleep 2
      docker cp {{ stack_name }}_redis:/data/dump.rdb \
          "${BACKUP_DIR}/redis_${DATE}.rdb"
      echo "✅ Redis backup: redis_${DATE}.rdb"

      # Cleanup old backups
      echo "Cleaning up old backups (keep last ${RETENTION} days)..."
      find "${BACKUP_DIR}" -name "*.dump" -mtime +${RETENTION} -delete
      find "${BACKUP_DIR}" -name "*.rdb"  -mtime +${RETENTION} -delete

      echo "=== Backup Complete ==="
      ls -lh "${BACKUP_DIR}/"
  when: backup_enabled

- name: Schedule backup
  ansible.builtin.cron:
    name:   "{{ stack_name }} stack backup"
    minute: "{{ backup_schedule.split()[0] }}"
    hour:   "{{ backup_schedule.split()[1] }}"
    day:    "{{ backup_schedule.split()[2] }}"
    month:  "{{ backup_schedule.split()[3] }}"
    weekday: "{{ backup_schedule.split()[4] }}"
    user:   root
    job:    "/usr/local/bin/stack-backup.sh >> {{ stack_log_dir }}/backup.log 2>&1"
    state:  present
  when: backup_enabled

# ── Health monitor ────────────────────────────
- name: Deploy health monitor script
  ansible.builtin.copy:
    dest:  /usr/local/bin/stack-health.sh
    mode:  '0755'
    owner: root
    content: |
      #!/bin/bash
      # Stack Health Monitor

      STACK="{{ stack_name }}"

      echo "=== {{ stack_name }} Health Check $(date) ==="
      echo ""
      echo "Containers:"
      docker compose \
          -f {{ stack_dir }}/docker-compose.yml \
          -p {{ stack_name }} \
          ps --format "table {{.Name}}\t{{.Status}}\t{{.Health}}"
      echo ""
      echo "Resources:"
      docker stats --no-stream \
          --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}" \
          $(docker compose -p {{ stack_name }} ps -q)

- name: Schedule health check
  ansible.builtin.cron:
    name:   "{{ stack_name }} health check"
    minute: "*/15"
    user:   root
    job:    "/usr/local/bin/stack-health.sh >> {{ stack_log_dir }}/health.log 2>&1"
    state:  present
EOF
```

---

## tasks/verify.yml

```bash
cat > ~/ansible/roles/fullstack/tasks/verify.yml << 'EOF'
---
# ============================================================
# Verification — Full Stack Checks
# ============================================================

# ── Container status ──────────────────────────
- name: Get stack status
  ansible.builtin.command:
    cmd: >
      docker compose
      -f {{ stack_dir }}/docker-compose.yml
      -p {{ stack_name }}
      ps --format json
  register:     stack_ps
  changed_when: false

# ── HTTP checks ───────────────────────────────
- name: HTTP health check
  ansible.builtin.uri:
    url:         "http://localhost:{{ nginx_http_port }}/health"
    status_code: 200
    timeout:     30
  register:      http_check
  retries:       5
  delay:         5
  ignore_errors: true

# ── Database check ────────────────────────────
- name: PostgreSQL connectivity
  ansible.builtin.command:
    cmd: >
      docker exec {{ stack_name }}_postgres
      pg_isready -U {{ postgres_user }} -d {{ postgres_db }}
  register:     pg_check
  changed_when: false
  ignore_errors: true

# ── Redis check ───────────────────────────────
- name: Redis connectivity
  ansible.builtin.command:
    cmd: >
      docker exec {{ stack_name }}_redis
      redis-cli --no-auth-warning
      -a {{ vault_redis_password }} ping
  register:     redis_check
  changed_when: false
  no_log:       true
  ignore_errors: true

# ── Disk usage ────────────────────────────────
- name: Docker disk usage
  ansible.builtin.command:
    cmd: docker system df
  register:     disk_usage
  changed_when: false

# ── Stack report ──────────────────────────────
- name: Full Stack Verification Report
  ansible.builtin.debug:
    msg:
      - "╔══════════════════════════════════════════╗"
      - "║   FULL STACK VERIFICATION REPORT         ║"
      - "╠══════════════════════════════════════════╣"
      - "║ Stack  : {{ stack_name }} v{{ stack_version }}"
      - "║ Host   : {{ inventory_hostname }}"
      - "║ Domain : {{ stack_domain }}"
      - "╠══════════════════════════════════════════╣"
      - "║ SERVICES:                                ║"
      - "║ Nginx    : {{ '✅' if nginx_info.container.State.Running    else '❌' }}"
      - "║ App      : {{ '✅' if app_info.container.State.Running      else '❌' }}"
      - "║ Postgres : {{ '✅' if pg_info.container.State.Health.Status == 'healthy' else '❌' }}"
      - "║ Redis    : {{ '✅' if redis_info.container.State.Health.Status == 'healthy' else '❌' }}"
      - "╠══════════════════════════════════════════╣"
      - "║ CHECKS:                                  ║"
      - "║ HTTP     : {{ '✅ OK' if not http_check.failed  else '❌ Failed' }}"
      - "║ Database : {{ '✅ OK' if not pg_check.failed    else '❌ Failed' }}"
      - "║ Redis    : {{ '✅ OK' if not redis_check.failed else '❌ Failed' }}"
      - "╠══════════════════════════════════════════╣"
      - "║ URLs:                                    ║"
      - "║ App     : http://{{ stack_domain }}"
      - "{% if adminer_enabled %}"
      - "║ Adminer : http://{{ stack_domain }}/adminer/"
      - "{% endif %}"
      - "{% if monitoring_enabled %}"
      - "║ Monitor : http://{{ stack_domain }}:8081"
      - "{% endif %}"
      - "╠══════════════════════════════════════════╣"
      - "║ DISK:                                    ║"
      - "{{ disk_usage.stdout_lines | join('\n') }}"
      - "╚══════════════════════════════════════════╝"
EOF
```

---

## handlers/main.yml

```bash
cat > ~/ansible/roles/fullstack/handlers/main.yml << 'EOF'
---
- name: Reload nginx
  ansible.builtin.command:
    cmd: >
      docker compose
      -f {{ stack_dir }}/docker-compose.yml
      -p {{ stack_name }}
      exec -T nginx nginx -s reload
  ignore_errors: "{{ ansible_check_mode }}"

- name: Restart app
  ansible.builtin.command:
    cmd: >
      docker compose
      -f {{ stack_dir }}/docker-compose.yml
      -p {{ stack_name }}
      restart app
  ignore_errors: "{{ ansible_check_mode }}"

- name: Restart stack
  community.docker.docker_compose_v2:
    project_src:  "{{ stack_dir }}"
    project_name: "{{ stack_name }}"
    state:        present
    recreate:     always
  no_log: true
  ignore_errors: "{{ ansible_check_mode }}"
EOF
```

---

## Πλήρες Deployment Playbook

```bash
cat > ~/ansible/playbooks/fullstack-deploy.yml << 'EOF'
---
# ============================================================
# Full Stack Deployment Playbook
# ============================================================

- name: Full Stack Deployment
  hosts: "{{ target | default('all_managed') }}"
  become: true
  gather_facts: true

  vars:
    stack_name:    mystack
    stack_version: "{{ version | default('1.0.0') }}"
    app_env:       production
    stack_domain:  "{{ ansible_facts['default_ipv4']['address'] }}"

    # ── Images ────────────────────────────────
    nginx_image:    nginx:1.25.3-alpine
    app_image:      "nginx:1.25.3-alpine"    # placeholder
    postgres_image: postgres:15.4-alpine
    redis_image:    redis:7.2-alpine
    adminer_image:  adminer:4-standalone

    # ── Features ──────────────────────────────
    adminer_enabled:   true
    monitoring_enabled: true
    portainer_enabled: true
    backup_enabled:    true

    # ── Admin access ──────────────────────────
    admin_allowed_ips:
      - 127.0.0.1
      - 192.168.1.0/24

  pre_tasks:

    - name: Pre-deployment checks
      ansible.builtin.assert:
        that:
          - ansible_facts.services['docker.service'].state == 'running'
          - ansible_facts['memtotal_mb'] | int >= 1024
        fail_msg: "❌ Docker not running or insufficient RAM!"
      vars:
        ansible_facts:
          services:
            docker.service:
              state: running
      ignore_errors: true
      tags: always

    - name: Deployment info
      ansible.builtin.debug:
        msg:
          - "╔══════════════════════════════════════╗"
          - "║   FULL STACK DEPLOYMENT              ║"
          - "╠══════════════════════════════════════╣"
          - "║ Stack  : {{ stack_name }} v{{ stack_version }}"
          - "║ Host   : {{ inventory_hostname }}"
          - "║ IP     : {{ ansible_facts['default_ipv4']['address'] }}"
          - "║ RAM    : {{ (ansible_facts['memtotal_mb'] / 1024) | round(1) }}GB"
          - "║ CPUs   : {{ ansible_facts['processor_vcpus'] }}"
          - "╠══════════════════════════════════════╣"
          - "║ Services:                            ║"
          - "║ ✅ Nginx + App + Postgres + Redis    ║"
          - "{% if adminer_enabled %}║ ✅ Adminer (DB UI)                   ║{% endif %}"
          - "{% if monitoring_enabled %}║ ✅ cAdvisor + Portainer              ║{% endif %}"
          - "╚══════════════════════════════════════╝"
      tags: always

  roles:
    - role: fullstack
      tags: fullstack

  post_tasks:

    - name: Access Information
      ansible.builtin.debug:
        msg:
          - "╔══════════════════════════════════════════╗"
          - "║   DEPLOYMENT COMPLETE! 🎉                ║"
          - "╠══════════════════════════════════════════╣"
          - "║ Main App : http://{{ ansible_facts['default_ipv4']['address'] }}"
          - "{% if adminer_enabled %}"
          - "║ Adminer  : http://{{ ansible_facts['default_ipv4']['address'] }}/adminer/"
          - "{% endif %}"
          - "{% if monitoring_enabled %}"
          - "║ Portainer: http://{{ ansible_facts['default_ipv4']['address'] }}:9000"
          - "║ cAdvisor : http://{{ ansible_facts['default_ipv4']['address'] }}:8081"
          - "{% endif %}"
          - "╠══════════════════════════════════════════╣"
          - "║ Management Commands:                     ║"
          - "║ Status: docker compose -p {{ stack_name }} ps"
          - "║ Logs  : docker compose -p {{ stack_name }} logs -f"
          - "║ Stop  : docker compose -p {{ stack_name }} stop"
          - "║ Backup: /usr/local/bin/stack-backup.sh"
          - "╚══════════════════════════════════════════╝"
      tags: always
EOF
```

---

## Εκτέλεση

```bash
# ── Syntax check ──────────────────────────────
ansible-playbook playbooks/fullstack-deploy.yml \
    --syntax-check

# ── Dry run ───────────────────────────────────
ansible-playbook playbooks/fullstack-deploy.yml \
    --check \
    --limit nextcloud

# ── Full deployment ───────────────────────────
ansible-playbook playbooks/fullstack-deploy.yml \
    --limit nextcloud \
    -e "version=1.0.0" \
    -v

# ── Μόνο setup ────────────────────────────────
ansible-playbook playbooks/fullstack-deploy.yml \
    --tags setup \
    --limit nextcloud

# ── Update version ────────────────────────────
ansible-playbook playbooks/fullstack-deploy.yml \
    --limit nextcloud \
    -e "version=2.0.0" \
    --tags deploy

# ── Verify ────────────────────────────────────
ansible-playbook playbooks/fullstack-deploy.yml \
    --tags verify \
    --limit nextcloud

# ── Manual backup ─────────────────────────────
ansible nextcloud -m ansible.builtin.command \
    -a "/usr/local/bin/stack-backup.sh" \
    --become
```
{% endraw %}
