# Ansible Complete Study Guide
---

## Table of Contents

1. [Ansible Fundamentals & Architecture](#module-1-ansible-fundamentals--architecture)
2. [Inventory Management](#module-2-inventory-management)
3. [Playbooks & YAML Basics](#module-3-playbooks--yaml-basics)
4. [Variables & Facts](#module-4-variables--facts)
5. [Modules & Ad-Hoc Commands](#module-5-modules--ad-hoc-commands)
6. [Conditionals, Loops & Handlers](#module-6-conditionals-loops--handlers)
7. [Roles & Ansible Galaxy](#module-7-roles--ansible-galaxy)
8. [Templates (Jinja2)](#module-8-templates-jinja2)
9. [Ansible Vault (Secrets Management)](#module-9-ansible-vault-secrets-management)
10. [Error Handling & Idempotency](#module-10-error-handling--idempotency)
11. [Ansible with Cloud (Dynamic Inventory)](#module-11-ansible-with-cloud-dynamic-inventory)
12. [Ansible Collections & Modularity](#module-12-ansible-collections--modularity)
13. [AWX / Ansible Automation Platform (Tower)](#module-13-awx--ansible-automation-platform-tower)
14. [Performance & Best Practices](#module-14-performance--best-practices)

---

## MODULE 1: Ansible Fundamentals & Architecture

### 1.1 What is Ansible, and how is it different from Puppet/Chef?

Ansible is an open-source, agentless automation tool for configuration management, application deployment, and orchestration — using SSH (or WinRM for Windows) to connect to managed nodes, requiring no agent installed on target systems.

| Ansible | Puppet / Chef |
|---|---|
| Agentless (SSH/WinRM) | Requires an agent on every managed node |
| Push-based (control node pushes changes) | Pull-based (agents pull config periodically) |
| YAML (declarative, human-readable) | Puppet DSL / Ruby (Chef) |
| Simple to start, minimal infra | Requires a master server + agent infrastructure |

---

### 1.2 Ansible Architecture

```
                Control Node
        (Ansible installed here — YOUR machine
         or a CI/CD runner, no daemon required)
                     │
             SSH / WinRM (agentless)
                     │
        ┌────────────┼────────────┐
   Managed Node A  Managed Node B  Managed Node C
   (no agent installed — just needs Python present)
```

- **Control Node** — the machine where Ansible is installed and playbooks are run from.
- **Managed Nodes** — target servers being configured; only need Python (for most modules) and SSH access.
- **Inventory** — the list of managed nodes Ansible operates on.
- **Modules** — units of work Ansible executes on managed nodes (e.g., `apt`, `copy`, `service`).
- **Playbooks** — YAML files defining a set of tasks to run against hosts.

---

### 1.3 Push vs Pull Model — Why Ansible Chose Push

Ansible's push model means changes propagate immediately when you run a playbook (rather than waiting for the next agent pull cycle), giving predictable, on-demand execution — at the tradeoff of needing network connectivity to every managed node at execution time (vs. pull-based agents that can apply configuration even if briefly disconnected from the control plane during the actual push).

---

### 1.4 Installing Ansible

```bash
# On a control node (e.g., Ubuntu)
sudo apt update && sudo apt install -y ansible

# Or via pip (recommended for version pinning)
pip install ansible --break-system-packages

ansible --version
```

---

### 1.5 Ansible Configuration File (`ansible.cfg`)

Ansible reads configuration in this precedence order: `ANSIBLE_CONFIG` env var → `./ansible.cfg` (current directory) → `~/.ansible.cfg` → `/etc/ansible/ansible.cfg`.

```ini
[defaults]
inventory = ./inventory/hosts.ini
remote_user = ansible
host_key_checking = False
retry_files_enabled = False
forks = 20

[privilege_escalation]
become = True
become_method = sudo
```

---

## MODULE 2: Inventory Management

### 2.1 Static Inventory (INI format)

```ini
[webservers]
web1.example.com
web2.example.com ansible_host=192.168.1.20

[dbservers]
db1.example.com ansible_user=dbadmin

[production:children]
webservers
dbservers

[webservers:vars]
http_port=80
```

```bash
ansible-inventory -i hosts.ini --list
ansible all -i hosts.ini -m ping
```

---

### 2.2 Static Inventory (YAML format)

```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
          ansible_host: 192.168.1.20
      vars:
        http_port: 80
    dbservers:
      hosts:
        db1.example.com:
          ansible_user: dbadmin
```

---

### 2.3 Host and Group Variables (`host_vars` / `group_vars`)

```
inventory/
├── hosts.ini
├── group_vars/
│   ├── webservers.yml
│   └── all.yml
└── host_vars/
    └── web1.example.com.yml
```

Ansible automatically loads variables from these directories based on the host/group name — no explicit `include` needed, keeping inventory data organized and separated from playbook logic.

---

### 2.4 Patterns for Targeting Hosts

```bash
ansible webservers -m ping                    # target a group
ansible "webservers:&dbservers" -m ping        # intersection (hosts in both groups)
ansible "webservers:!web2.example.com" -m ping  # exclude a specific host
ansible "web*" -m ping                          # wildcard pattern
ansible all --limit "web1.example.com,web2.example.com" -m ping
```

---

## MODULE 3: Playbooks & YAML Basics

### 3.1 Anatomy of a Playbook

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true
  vars:
    http_port: 8080
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: true

    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: true
```

```bash
ansible-playbook -i inventory/hosts.ini site.yml
ansible-playbook site.yml --check          # dry-run
ansible-playbook site.yml --diff           # show file changes
ansible-playbook site.yml --limit webservers
```

---

### 3.2 Common YAML Pitfalls in Ansible

- Indentation errors (YAML is whitespace-sensitive — mixing tabs and spaces breaks parsing).
- Forgetting the `---` document start marker (optional but conventional).
- Boolean ambiguity — `yes`/`no`/`true`/`false` all work, but mixing styles inconsistently across a codebase hurts readability.
- Quoting strings that start with special characters (e.g., `{{ }}` at the very start of a value) — always quote Jinja2 expressions used as full string values.

```yaml
# Wrong: unquoted Jinja2 at start of line can break YAML parsing
port: {{ http_port }}

# Correct
port: "{{ http_port }}"
```

---

### 3.3 Multiple Plays in One Playbook

```yaml
---
- name: Configure web tier
  hosts: webservers
  tasks:
    - name: Install nginx
      apt: { name: nginx, state: present }

- name: Configure database tier
  hosts: dbservers
  tasks:
    - name: Install postgresql
      apt: { name: postgresql, state: present }
```

Each play targets a different group of hosts and runs sequentially — useful for orchestrating a multi-tier deployment in a single playbook run.

---

## MODULE 4: Variables & Facts

### 4.1 Variable Precedence (High to Low, Simplified)

```
1. Extra vars (-e on command line)          ← highest precedence
2. Task vars
3. Block vars
4. Role and include vars
5. Play vars
6. Host facts / registered vars
7. host_vars / group_vars
8. Role defaults                             ← lowest precedence
```

This precedence order is one of the most commonly tested Ansible interview topics — extra vars (`-e`) always win, and role `defaults/main.yml` values are the easiest to override.

```bash
ansible-playbook site.yml -e "http_port=9090"
```

---

### 4.2 Ansible Facts (Auto-Discovered System Information)

```yaml
- name: Show OS family
  debug:
    msg: "{{ ansible_facts['os_family'] }}"

- name: Show total memory
  debug:
    msg: "{{ ansible_facts['memtotal_mb'] }}"
```

```bash
ansible web1.example.com -m setup            # dump all facts
ansible web1.example.com -m setup -a "filter=ansible_distribution*"
```

Disable fact gathering for faster runs when facts aren't needed:
```yaml
- hosts: webservers
  gather_facts: false
```

---

### 4.3 Registering Task Output as a Variable

```yaml
- name: Check if a file exists
  stat:
    path: /etc/myapp/config.yml
  register: config_check

- name: Create config only if missing
  copy:
    src: config.yml
    dest: /etc/myapp/config.yml
  when: not config_check.stat.exists
```

---

### 4.4 Using `set_fact` for Computed Variables

```yaml
- name: Compute a derived value
  set_fact:
    full_url: "https://{{ inventory_hostname }}:{{ http_port }}/health"
```

---

## MODULE 5: Modules & Ad-Hoc Commands

### 5.1 Ad-Hoc Commands

```bash
ansible webservers -m ping
ansible webservers -m command -a "uptime"
ansible webservers -m shell -a "df -h | grep /dev/sda1"
ansible webservers -m copy -a "src=./app.conf dest=/etc/app.conf"
ansible webservers -m service -a "name=nginx state=restarted" --become
```

Ad-hoc commands are ideal for quick, one-off tasks; playbooks are for anything repeatable/version-controlled.

---

### 5.2 `command` vs `shell` vs `raw` Modules

| Module | Behavior |
|---|---|
| `command` | Runs command directly, no shell processing (no pipes/redirects/env vars) — safer, preferred default |
| `shell` | Runs through `/bin/sh`, supports pipes/redirects/variables — use only when genuinely needed |
| `raw` | Bypasses the Python module system entirely — used for bootstrapping hosts without Python installed |

```yaml
- name: Safer - no shell injection risk
  command: /usr/bin/systemctl restart nginx

- name: Needed - uses a pipe
  shell: "ps aux | grep nginx | wc -l"
```

---

### 5.3 Commonly Used Modules

```yaml
- name: Package management
  apt: { name: nginx, state: present }       # Debian/Ubuntu
  # yum / dnf for RHEL-based systems

- name: File management
  copy: { src: file.txt, dest: /tmp/file.txt, mode: '0644' }

- name: Template rendering
  template: { src: config.j2, dest: /etc/app/config.yml }

- name: Service management
  service: { name: nginx, state: started, enabled: true }

- name: User management
  user: { name: deploy, groups: sudo, shell: /bin/bash }

- name: Line in file
  lineinfile: { path: /etc/ssh/sshd_config, regexp: '^PermitRootLogin', line: 'PermitRootLogin no' }
```

---

### 5.4 Idempotency — The Core Ansible Principle

A module is idempotent if running it multiple times produces the same end state without unintended side effects — e.g., `apt: name=nginx state=present` only installs nginx if it's not already installed, reporting "ok" (not "changed") on subsequent runs.

**Interview trap:** `command`/`shell` modules are **not** idempotent by default (they run every time regardless of current state) — use `creates`/`removes` arguments, or `when` conditions checked via `register`, to make them idempotent manually.

```yaml
- name: Only run if the marker file doesn't exist
  command: /opt/setup.sh
  args:
    creates: /opt/.setup_complete
```

---

## MODULE 6: Conditionals, Loops & Handlers

### 6.1 Conditionals with `when`

```yaml
- name: Install nginx only on Debian-based systems
  apt:
    name: nginx
    state: present
  when: ansible_facts['os_family'] == "Debian"

- name: Restart service if config changed and env is production
  service:
    name: myapp
    state: restarted
  when:
    - config_result.changed
    - environment == "production"
```

---

### 6.2 Loops

```yaml
- name: Install multiple packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - git
    - curl

- name: Create multiple users
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
  loop:
    - { name: 'alice', groups: 'admin' }
    - { name: 'bob', groups: 'developers' }
```

---

### 6.3 Handlers and `notify`

Handlers run only when notified by a task that reports `changed`, and only run **once** at the end of the play even if notified multiple times — ideal for actions like "restart service" that should only happen once regardless of how many config files changed.

```yaml
tasks:
  - name: Update nginx config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: Restart nginx

  - name: Update nginx site config
    template:
      src: site.conf.j2
      dest: /etc/nginx/sites-available/default
    notify: Restart nginx

handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
```

---

### 6.4 Blocks and Error Handling

```yaml
- name: Attempt risky operation with rescue
  block:
    - name: Try to update the application
      command: /opt/deploy.sh
  rescue:
    - name: Roll back on failure
      command: /opt/rollback.sh
  always:
    - name: Always send a notification
      debug:
        msg: "Deployment attempt finished"
```

`block`/`rescue`/`always` mirrors try/catch/finally semantics — `rescue` only runs if a task in `block` fails, and `always` runs regardless of outcome.

---

## MODULE 7: Roles & Ansible Galaxy

### 7.1 Role Directory Structure

```
roles/
└── nginx/
    ├── defaults/main.yml      # lowest-precedence default variables
    ├── vars/main.yml          # higher-precedence variables
    ├── tasks/main.yml         # the main list of tasks
    ├── handlers/main.yml      # handlers notified by tasks
    ├── templates/nginx.conf.j2
    ├── files/                 # static files copied as-is
    ├── meta/main.yml          # role dependencies, metadata
    └── README.md
```

```bash
ansible-galaxy init roles/nginx     # scaffold a new role structure
```

---

### 7.2 Using Roles in a Playbook

```yaml
---
- hosts: webservers
  roles:
    - nginx
    - { role: app, app_version: "2.1.0" }

# Or the modern include-based syntax for more control over ordering
- hosts: webservers
  tasks:
    - name: Apply nginx role
      include_role:
        name: nginx
```

---

### 7.3 Ansible Galaxy — Installing Community Roles/Collections

```bash
ansible-galaxy install geerlingguy.nginx
ansible-galaxy collection install community.general
```

```yaml
# requirements.yml — version-pin dependencies for reproducibility
roles:
  - name: geerlingguy.nginx
    version: "3.1.4"
collections:
  - name: community.general
    version: "8.0.0"
```

```bash
ansible-galaxy install -r requirements.yml
```

---

### 7.4 Role Dependencies (`meta/main.yml`)

```yaml
dependencies:
  - role: common
  - role: firewall
    vars:
      allowed_ports: [80, 443]
```

Dependencies listed here run automatically before the role's own tasks — useful for enforcing that prerequisite roles (like a `common` baseline hardening role) always execute first.

---

## MODULE 8: Templates (Jinja2)

### 8.1 Basic Templating

```jinja2
{# templates/nginx.conf.j2 #}
server {
    listen {{ http_port }};
    server_name {{ inventory_hostname }};

    location / {
        proxy_pass http://{{ backend_host }}:{{ backend_port }};
    }
}
```

```yaml
- name: Deploy nginx config from template
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/default
  notify: Restart nginx
```

---

### 8.2 Jinja2 Filters

```jinja2
{{ my_list | join(", ") }}
{{ my_string | upper }}
{{ my_dict | to_json }}
{{ undefined_var | default("fallback_value") }}
{{ my_list | length }}
{{ items | selectattr("enabled", "equalto", true) | list }}
```

---

### 8.3 Conditionals and Loops Inside Templates

```jinja2
{% for server in upstream_servers %}
upstream backend {
    server {{ server.host }}:{{ server.port }};
}
{% endfor %}

{% if enable_ssl %}
listen 443 ssl;
{% else %}
listen 80;
{% endif %}
```

---

## MODULE 9: Ansible Vault (Secrets Management)

### 9.1 Encrypting Files and Variables

```bash
ansible-vault create secrets.yml          # create a new encrypted file
ansible-vault edit secrets.yml            # edit an existing encrypted file
ansible-vault encrypt existing_vars.yml   # encrypt an existing plaintext file
ansible-vault decrypt secrets.yml         # decrypt permanently (careful!)
ansible-vault view secrets.yml            # view without editing
```

```bash
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file=~/.vault_pass.txt
```

---

### 9.2 Encrypting Individual Variables (Not Whole Files)

```bash
ansible-vault encrypt_string 'supersecretpassword' --name 'db_password'
```

```yaml
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          663864396235633965663963343163336332...
```

This allows a variables file to remain readable/diffable in Git while keeping only the specific sensitive value encrypted — a common best practice over encrypting entire files.

---

### 9.3 Multiple Vault Passwords (Vault IDs)

```bash
ansible-vault encrypt secrets_prod.yml --vault-id prod@prompt
ansible-vault encrypt secrets_dev.yml --vault-id dev@prompt
ansible-playbook site.yml --vault-id prod@~/.vault_pass_prod.txt --vault-id dev@~/.vault_pass_dev.txt
```

Vault IDs let you use different passwords for different environments (dev/staging/prod), so a leaked dev vault password doesn't compromise production secrets.

---

### 9.4 Integrating Ansible Vault with External Secret Managers

For enterprise setups, many teams avoid storing even encrypted secrets in Git entirely, instead using a lookup plugin to fetch secrets at runtime:

```yaml
- name: Fetch secret from HashiCorp Vault
  debug:
    msg: "{{ lookup('community.hashi_vault.hashi_vault', 'secret=secret/data/myapp:password') }}"

- name: Fetch secret from AWS Secrets Manager
  debug:
    msg: "{{ lookup('amazon.aws.aws_secret', 'my-secret-name') }}"
```

---

## MODULE 10: Error Handling & Idempotency

### 10.1 `ignore_errors` and `failed_when`

```yaml
- name: Continue even if this task fails
  command: /opt/optional_script.sh
  ignore_errors: true

- name: Custom failure condition
  command: /opt/healthcheck.sh
  register: result
  failed_when: "'ERROR' in result.stdout"
```

---

### 10.2 `changed_when` — Controlling Idempotency Reporting

```yaml
- name: Run a command that never actually changes anything
  command: /opt/status_check.sh
  register: result
  changed_when: false     # prevents this from ever reporting as "changed"
```

This matters for accurate reporting/handler triggering — a `command`/`shell` task always reports "changed" by default regardless of whether it actually modified anything, which can cause unwanted handler notifications unless corrected with `changed_when`.

---

### 10.3 Debugging Playbooks

```bash
ansible-playbook site.yml -vvv                # increasing verbosity levels (-v to -vvvv)
ansible-playbook site.yml --syntax-check       # check YAML/playbook syntax only
ansible-playbook site.yml --list-tasks         # preview what tasks would run
ansible-playbook site.yml --list-hosts         # preview which hosts would be targeted
ansible-playbook site.yml --step               # confirm each task interactively
ansible-playbook site.yml --start-at-task="Install nginx"  # resume from a specific task
```

```yaml
- name: Debug a variable's value
  debug:
    var: my_variable

- name: Debug with a custom message
  debug:
    msg: "The value is {{ my_variable }}"
```

---

### 10.4 `any_errors_fatal` and `max_fail_percentage`

```yaml
- hosts: webservers
  any_errors_fatal: true      # stop the entire run immediately if any host fails
  max_fail_percentage: 30     # abort if more than 30% of hosts fail
  tasks:
    - name: Deploy application
      command: /opt/deploy.sh
```

By default, Ansible continues running a play against remaining hosts even if one host fails — these settings let you enforce stricter, safer rollout behavior for production deployments.

---

## MODULE 11: Ansible with Cloud (Dynamic Inventory)

### 11.1 Why Dynamic Inventory?

Static inventory files become unmanageable in cloud environments where instances are created/destroyed frequently (Auto Scaling Groups, ephemeral containers) — dynamic inventory plugins query the cloud provider's API in real time to build the current, accurate inventory automatically.

```yaml
# inventory/aws_ec2.yml
plugin: amazon.aws.aws_ec2
regions:
  - ap-south-1
filters:
  tag:Environment: production
keyed_groups:
  - key: tags.Role
    prefix: role
```

```bash
ansible-inventory -i inventory/aws_ec2.yml --graph
ansible-playbook -i inventory/aws_ec2.yml site.yml
```

---

### 11.2 GCP Dynamic Inventory

```yaml
# inventory/gcp.yml
plugin: google.cloud.gcp_compute
projects:
  - my-gcp-project
zones:
  - asia-south1-a
auth_kind: application
```

---

### 11.3 Azure Dynamic Inventory

```yaml
# inventory/azure_rm.yml
plugin: azure.azcollection.azure_rm
include_vm_resource_groups:
  - my-resource-group
```

---

### 11.4 Cloud Modules Example (AWS EC2 Provisioning + Configuration in One Playbook)

```yaml
- name: Provision and configure an EC2 instance
  hosts: localhost
  tasks:
    - name: Launch EC2 instance
      amazon.aws.ec2_instance:
        name: my-web-server
        instance_type: t3.medium
        image_id: ami-0abc123
        region: ap-south-1
        wait: true
      register: ec2_result

    - name: Add new instance to in-memory inventory
      add_host:
        hostname: "{{ ec2_result.instances[0].public_ip_address }}"
        groups: new_instances

- name: Configure the newly launched instance
  hosts: new_instances
  tasks:
    - name: Install nginx
      apt: { name: nginx, state: present }
```

This pattern (provision, then immediately configure in the same run using `add_host`) is a commonly-asked practical Ansible+cloud interview scenario.

---

## MODULE 12: Ansible Collections & Modularity

### 12.1 What is an Ansible Collection?

A distributable, versioned package bundling roles, modules, plugins, and playbooks together — the modern packaging format for sharing Ansible content (replacing the older, less structured role-only distribution model).

```bash
ansible-galaxy collection install amazon.aws
ansible-galaxy collection list
```

```yaml
# Using a module from a collection explicitly
- name: Create an S3 bucket
  amazon.aws.s3_bucket:
    name: my-bucket
    region: ap-south-1
    state: present
```

---

### 12.2 Building a Custom Collection

```bash
ansible-galaxy collection init myorg.mycollection
```

```
myorg/mycollection/
├── galaxy.yml
├── plugins/
│   └── modules/
├── roles/
└── README.md
```

Organizations commonly build internal collections to package and version their standardized roles/modules for reuse across teams, similar to how Terraform modules are shared internally.

---

## MODULE 13: AWX / Ansible Automation Platform (Tower)

### 13.1 What is AWX / Ansible Automation Platform?

AWX (open-source) / Ansible Automation Platform (Red Hat's commercial offering, formerly "Tower") provides a web UI, REST API, RBAC, scheduling, and centralized logging on top of core Ansible — turning command-line playbook execution into a governed, auditable, team-oriented automation platform.

```
Web UI / REST API
        │
Job Templates (playbook + inventory + credentials bundle)
        │
Execution Environment (containerized runtime with Ansible + collections)
        │
Managed Nodes
```

---

### 13.2 Key AWX Concepts

| Concept | Purpose |
|---|---|
| Project | Links to a Git repo containing playbooks |
| Inventory | Static or dynamic inventory sources |
| Credentials | Securely stored SSH keys, cloud credentials, Vault passwords |
| Job Template | A reusable, launchable combination of playbook + inventory + credentials + variables |
| Workflow Template | Chains multiple job templates together with conditional branching |
| RBAC | Fine-grained access control over who can view/launch/edit specific templates |

---

### 13.3 Why Use AWX/Tower Instead of Running `ansible-playbook` from a Laptop?

- Centralized, audited execution logs (who ran what, when, with what result).
- Credentials are stored securely and never exposed to the person launching a job (they don't need the actual SSH key/password).
- RBAC — grant a junior engineer permission to launch a specific job template without giving them raw SSH access or the ability to edit the underlying playbook.
- Scheduling and Workflow orchestration across multiple playbooks/teams.

---

### 13.4 Execution Environments

Execution Environments are containerized runtime images bundling a specific Ansible version, Python dependencies, and collections — ensuring consistent, reproducible playbook execution regardless of which AWX node happens to run the job, and enabling different projects to use different, isolated dependency sets without conflicts.

---

## MODULE 14: Performance & Best Practices

### 14.1 Speeding Up Playbook Execution

```ini
# ansible.cfg
[defaults]
forks = 20                        # increase parallelism (default is 5)
gathering = smart                 # cache facts, avoid redundant gathering
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts_cache
fact_caching_timeout = 86400

[ssh_connection]
pipelining = True                 # reduces the number of SSH operations per task
```

**`pipelining = True`** is one of the single biggest performance wins — it reduces SSH round-trips by executing modules over a single SSH connection rather than transferring and executing them as separate steps, but requires `requiretty` to be disabled in `/etc/sudoers` on managed nodes.

---

### 14.2 Best Practices Checklist

- Use roles to organize reusable logic instead of one giant monolithic playbook.
- Pin collection/role versions in `requirements.yml` for reproducible environments.
- Never store plaintext secrets — use Ansible Vault or an external secrets manager lookup.
- Prefer purpose-built modules (`apt`, `copy`, `template`) over `shell`/`command` wherever a module exists, for proper idempotency and clearer intent.
- Use `--check` (dry-run) and `--diff` in CI pipelines before applying changes to production.
- Tag tasks (`tags:`) to allow selective execution (`--tags`, `--skip-tags`) without needing separate playbooks for partial runs.
- Keep inventory and variables separated cleanly using `group_vars`/`host_vars` rather than hardcoding values inside playbooks.

```yaml
- name: Install packages
  apt:
    name: "{{ item }}"
  loop: [nginx, git]
  tags: [packages, setup]
```

```bash
ansible-playbook site.yml --tags packages
ansible-playbook site.yml --skip-tags setup
```

---

### 14.3 Testing Ansible Playbooks

- **`ansible-lint`** — static analysis catching style issues, deprecated syntax, and common mistakes before they cause runtime problems.
- **Molecule** — a testing framework for roles, spinning up ephemeral test environments (Docker/cloud) to verify a role behaves correctly and idempotently.

```bash
ansible-lint site.yml
molecule test    # inside a role directory with a molecule/ scenario configured
```

---

## Summary Cheat-Sheet: Most-Used Ansible Commands

```bash
# Ad-hoc
ansible all -m ping
ansible webservers -m shell -a "uptime"

# Playbooks
ansible-playbook site.yml --check --diff
ansible-playbook site.yml --limit webservers --tags deploy

# Inventory
ansible-inventory -i inventory/hosts.ini --graph
ansible-inventory -i inventory/aws_ec2.yml --list

# Vault
ansible-vault create secrets.yml
ansible-vault encrypt_string 'secret' --name 'my_var'

# Galaxy / Collections
ansible-galaxy install -r requirements.yml
ansible-galaxy collection install community.general

# Debugging
ansible-playbook site.yml -vvv --syntax-check
ansible-playbook site.yml --list-tasks --list-hosts

# Linting/Testing
ansible-lint site.yml
molecule test
```

---
