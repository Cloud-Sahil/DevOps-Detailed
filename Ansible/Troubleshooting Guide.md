# Ansible Troubleshooting Guide

> A modular, practical troubleshooting reference for Ansible — organized by symptom/domain, with diagnostic commands, root-cause checklists, and resolution steps for connectivity, playbook execution, variables/templating, modules, performance, Vault, dynamic inventory, and AWX/Tower.

---

## Table of Contents

1. [General Troubleshooting Methodology](#module-1-general-troubleshooting-methodology)
2. [Connection Issues (SSH/WinRM)](#module-2-connection-issues-sshwinrm)
3. [Playbook Execution Errors](#module-3-playbook-execution-errors)
4. [Variable & Templating Issues](#module-4-variable--templating-issues)
5. [Module-Specific Issues](#module-5-module-specific-issues)
6. [Idempotency & Handler Issues](#module-6-idempotency--handler-issues)
7. [Performance Issues](#module-7-performance-issues)
8. [Ansible Vault Issues](#module-8-ansible-vault-issues)
9. [Dynamic Inventory & Cloud Issues](#module-9-dynamic-inventory--cloud-issues)
10. [Role & Collection Issues](#module-10-role--collection-issues)
11. [AWX / Tower Issues](#module-11-awx--tower-issues)
12. [CI/CD Integration Issues](#module-12-cicd-integration-issues)

---

## MODULE 1: General Troubleshooting Methodology

### 1.1 The Layered Troubleshooting Approach

```
Playbook run fails / behaves unexpectedly
        │
1. Increase verbosity (-vvv) to see raw module input/output
        │
2. Check connectivity independently (ansible -m ping)
        │
3. Check syntax (--syntax-check) before assuming a logic bug
        │
4. Isolate the failing task (--start-at-task, --step, or a minimal test playbook)
        │
5. Check variable resolution (debug module, ansible-inventory)
        │
6. Check target host state directly (SSH in manually, inspect logs)
        │
7. Fix, re-run with --check first, then apply, document
```

---

### 1.2 Essential Diagnostic Commands to Always Have Ready

```bash
ansible --version                                    # Ansible + Python + config file in use
ansible-config dump --only-changed                    # show non-default config settings
ansible-inventory -i inventory/hosts.ini --list        # confirm resolved inventory/vars
ansible all -m ping -i inventory/hosts.ini              # basic connectivity check
ansible-playbook site.yml --syntax-check                # catch YAML/syntax errors fast
ansible-playbook site.yml -vvv                           # maximum verbosity for deep debugging
```

---

## MODULE 2: Connection Issues (SSH/WinRM)

### 2.1 "UNREACHABLE" — Host Unreachable Errors

```bash
ansible web1.example.com -m ping -vvv
ssh -v ansible@web1.example.com     # test raw SSH manually, bypassing Ansible entirely
```

**Checklist:**
- Is the hostname/IP in inventory correct and resolvable (`nslookup`/`ping` at the OS level)?
- Is the SSH port correct (`ansible_port` if non-22)?
- Is the SSH user correct (`ansible_user`), and does the SSH key/password actually work when tested manually?
- Firewall/Security Group blocking port 22 from the control node's IP?

---

### 2.2 SSH Key Authentication Failing

```bash
ansible web1.example.com -m ping --private-key=~/.ssh/id_rsa -u ansible -vvv
```
Common causes: wrong private key path specified (or relying on ssh-agent when the key isn't loaded — `ssh-add ~/.ssh/id_rsa`), incorrect file permissions on the private key (must be `600`), or the public key not actually present in the target's `~/.ssh/authorized_keys`.

```bash
chmod 600 ~/.ssh/id_rsa
ssh-add ~/.ssh/id_rsa
ssh-add -l    # confirm the key is loaded
```

---

### 2.3 "Host key verification failed"

```
fatal: [web1.example.com]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: Host key verification failed."}
```
Occurs when the host's SSH key isn't in `~/.ssh/known_hosts` (or has changed, e.g., after a server rebuild). Fix properly by adding the correct key (`ssh-keyscan web1.example.com >> ~/.ssh/known_hosts`), or for ephemeral cloud/lab environments, set `host_key_checking = False` in `ansible.cfg` (never do this for production hosts where key verification is a genuine security control).

---

### 2.4 "sudo: a password is required" / Privilege Escalation Failures

```bash
ansible web1.example.com -m ping --become --ask-become-pass -vvv
```
Checklist: confirm `become_method` (default `sudo`) is correct for the target OS, the remote user actually has passwordless sudo configured (`visudo` → `NOPASSWD:ALL` for that user) if you're not passing `--ask-become-pass`, and that `become_user` defaults to `root` unless explicitly overridden.

```ini
# ansible.cfg
[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

---

### 2.5 WinRM Connection Failures (Windows Managed Nodes)

```bash
ansible windows_hosts -m win_ping -vvv
```
Checklist: confirm WinRM is actually enabled and listening on the Windows host (`winrm quickconfig`), the correct `ansible_connection: winrm` and `ansible_winrm_transport` (e.g., `ntlm`, `kerberos`, `credssp`) are set in inventory, and firewall allows the WinRM port (5985 HTTP / 5986 HTTPS).

```yaml
ansible_connection: winrm
ansible_winrm_transport: ntlm
ansible_port: 5986
ansible_winrm_server_cert_validation: ignore   # only for lab/self-signed certs
```

---

### 2.6 Connection Timeouts on Large Fleets

```ini
[defaults]
timeout = 30
forks = 20
```
If a small number of hosts in a large inventory are unreachable (e.g., decommissioned servers still listed), the default per-host connection timeout multiplied across many unreachable hosts can make an entire run appear to hang — increase `forks` for parallelism and audit/clean stale inventory entries rather than just raising the timeout indefinitely.

---

## MODULE 3: Playbook Execution Errors

### 3.1 YAML Syntax Errors

```bash
ansible-playbook site.yml --syntax-check
yamllint site.yml    # more detailed YAML-level linting
```
Common causes: mixed tabs/spaces, incorrect indentation (especially after adding/removing a list item), missing colons, or an unquoted string starting with `{{ }}` that YAML misinterprets as a flow-mapping.

---

### 3.2 "The task includes an option with an undefined variable"

```bash
ansible-playbook site.yml -vvv    # shows exactly which variable and where it was expected
```
Checklist: check for typos in the variable name, confirm the variable is actually defined in the expected scope (host_vars/group_vars/role defaults/extra-vars) using `ansible-inventory --list` or a `debug: var=hostvars[inventory_hostname]` task, and check variable precedence — a variable might be defined but overridden to an empty/undefined value at a higher-precedence layer.

```yaml
- name: Debug all resolved variables for a host
  debug:
    var: hostvars[inventory_hostname]
```

---

### 3.3 A Task Reports "changed" Every Single Run (Never Reaches Idempotent "ok")

Almost always a `command`/`shell` task without `changed_when`/`creates` — these modules report "changed" unconditionally on every execution regardless of actual system state change.

```yaml
- name: Fix — make this idempotent
  command: /opt/some_script.sh
  args:
    creates: /opt/.marker_file
```

---

### 3.4 Play Runs but Skips All Tasks Unexpectedly

```bash
ansible-playbook site.yml -vvv
ansible-inventory -i inventory/hosts.ini --host web1.example.com
```
Check `when` conditions evaluating to false due to an unexpected fact value (e.g., `ansible_facts['os_family']` returning something different than assumed — verify with `ansible -m setup`), or a host-limiting pattern (`--limit`) inadvertently excluding hosts you expected to be targeted.

---

### 3.5 "ERROR! the role/include ... was not found"

```bash
ansible-galaxy list                          # confirm the role is actually installed
ansible-config dump | grep ROLES_PATH        # check where Ansible is looking for roles
```
Common causes: role not installed (`ansible-galaxy install -r requirements.yml` never run), `roles_path` in `ansible.cfg` not pointing to where the role actually lives, or a typo in the role name in the playbook versus the actual directory name.

---

### 3.6 Playbook Hangs Indefinitely on a Specific Task

```bash
ansible-playbook site.yml -vvv --step
```
Common causes: a task waiting on interactive input the module doesn't expect (e.g., a `command` invoking a script that prompts for input with no `stdin` provided), an `async`/`poll` misconfiguration on a long-running task, or a genuine network hang reaching a downstream dependency the task calls out to — use `--step` to isolate exactly which task is stuck.

```yaml
- name: Long-running task run asynchronously with proper polling
  command: /opt/long_process.sh
  async: 600
  poll: 15
```

---

## MODULE 4: Variable & Templating Issues

### 4.1 Jinja2 Template Rendering Errors

```bash
ansible-playbook site.yml -vvv   # shows the exact Jinja2 error and template line
```
Common causes: referencing an undefined variable inside the template (use `| default('value')` filter defensively), incorrect filter syntax, or a missing `{% endif %}`/`{% endfor %}` closing tag.

```jinja2
{{ my_var | default('fallback') }}
```

---

### 4.2 Variable Has an Unexpected Value — Precedence Confusion

```bash
ansible-inventory -i inventory/hosts.ini --host web1.example.com
```
Walk through the precedence order deliberately (extra-vars → task vars → block vars → role/include vars → play vars → facts/registered vars → host_vars/group_vars → role defaults) to find where the unexpected value is actually coming from — a common trap is a `group_vars/all.yml` value silently overriding a more specific `host_vars` entry due to a misunderstanding of precedence order.

---

### 4.3 List/Dictionary Merging Not Behaving as Expected

By default, Ansible **replaces** variables entirely (a list/dict defined at a higher-precedence layer completely overwrites, not merges with, a lower-precedence one) — a common source of confusion for engineers expecting additive merging.

```ini
# ansible.cfg — enable hash merging for dictionaries specifically (not lists)
[defaults]
hash_behaviour = merge
```
Note: `hash_behaviour = merge` only affects dictionaries, not lists, and changing this globally can have wide-reaching effects — often better to explicitly combine variables using the `combine()` filter in the specific place it's needed rather than a global config change.

```jinja2
{{ base_dict | combine(override_dict) }}
```

---

### 4.4 `{{ }}` Values Not Being Substituted (Printed Literally)

Usually caused by using single quotes around the entire value in a context where Jinja2 templating isn't being invoked (e.g., inside a raw string in certain module arguments), or a missing `template` module usage where a `copy` was used instead (copy does not perform Jinja2 rendering, template does).

```yaml
# Wrong — copy does NOT render Jinja2
- copy: { src: config.j2, dest: /etc/app/config.yml }

# Correct — template DOES render Jinja2
- template: { src: config.j2, dest: /etc/app/config.yml }
```

---

## MODULE 5: Module-Specific Issues

### 5.1 `apt`/`yum` Module Fails with "Failed to lock apt/rpm database"

```yaml
- name: Wait for any other apt process to release the lock
  shell: while fuser /var/lib/dpkg/lock-frontend >/dev/null 2>&1; do sleep 1; done
```
Common cause: an unattended-upgrades process or another concurrent `apt`/`yum` operation holding the package manager lock — either wait for it, or (better) disable automatic OS-level package management processes on hosts managed by Ansible to avoid this race condition entirely.

---

### 5.2 `copy`/`template` Module Fails with Permission Denied

```bash
ansible web1.example.com -m command -a "ls -la /etc/app/" --become
```
Checklist: confirm `become: true` is set for the task (writing to a root-owned path without privilege escalation), check the destination directory actually exists (module won't auto-create parent directories by default), and verify `mode`/`owner`/`group` parameters aren't conflicting with restrictive existing permissions on the parent directory.

---

### 5.3 `service`/`systemd` Module Reports "Could not find the requested service"

```bash
ansible web1.example.com -m command -a "systemctl list-unit-files | grep myapp" --become
```
Common causes: service name typo, the service unit file genuinely doesn't exist yet (needs to be deployed via a `template`/`copy` task first, followed by `systemd: daemon_reload=true`), or using the `service` module (generic, less capable) instead of the more precise `systemd` module on systemd-based systems.

```yaml
- name: Reload systemd after adding a new unit file
  systemd:
    daemon_reload: true

- name: Now start the service
  systemd:
    name: myapp
    state: started
    enabled: true
```

---

### 5.4 `git` Module Fails with Authentication Errors

```bash
ansible web1.example.com -m command -a "ssh -T git@github.com" --become
```
Confirm the SSH key used for Git authentication is actually available to the user context the task runs as (often different from the user's own interactive shell environment, especially under `become`), and that `accept_hostkey: true` is set if the Git host's key isn't already trusted.

```yaml
- name: Clone a private repo
  git:
    repo: git@github.com:myorg/myrepo.git
    dest: /opt/myapp
    key_file: /home/deploy/.ssh/id_rsa
    accept_hostkey: true
```

---

### 5.5 Module Behaves Differently Across OS Distributions

```yaml
- name: OS-specific package name
  package:
    name: "{{ 'httpd' if ansible_facts['os_family'] == 'RedHat' else 'apache2' }}"
    state: present
```
Use the generic `package`/`service` modules (which delegate to the correct OS-specific backend automatically) combined with `ansible_facts` for OS-specific naming differences, rather than hardcoding `apt`/`yum` directly in cross-platform roles.

---

## MODULE 6: Idempotency & Handler Issues

### 6.1 Handler Never Fires Despite the Notifying Task Reporting "changed"

Checklist: confirm the handler name in `notify:` exactly matches the handler's `name:` field (case-sensitive, exact string match — a very common typo source), and confirm the handler is actually defined in the `handlers:` section of the same play (or an included/role handler file that's actually being loaded).

```bash
ansible-playbook site.yml -vvv   # shows "NOTIFIED HANDLER" lines confirming trigger
```

---

### 6.2 Handler Fires Multiple Times Instead of Once

By design, handlers should only run once at the end of the play regardless of how many tasks notify them — if you're seeing multiple executions, check whether you have multiple plays in the playbook each independently triggering the same-named handler (handlers are scoped per-play, not globally across an entire multi-play playbook run).

---

### 6.3 Forcing Handlers to Run Immediately (Not at End of Play)

```yaml
- name: Update config
  template: { src: app.conf.j2, dest: /etc/app.conf }
  notify: Restart app

- name: Flush handlers now instead of waiting until end of play
  meta: flush_handlers

- name: A later task that depends on the restart having already happened
  command: /opt/verify_restart.sh
```

---

### 6.4 A Task That Should Be Idempotent Keeps Reporting "changed"

```yaml
- name: Debug what the module thinks changed
  template:
    src: app.conf.j2
    dest: /etc/app.conf
  register: result
- debug: var=result.diff
```
Common causes: the template output includes a non-deterministic value (a timestamp, a randomly-ordered dict/list rendered inconsistently between runs), or line-ending differences (CRLF vs LF) between the rendered template and the existing file causing a diff every time.

---

## MODULE 7: Performance Issues

### 7.1 Playbook Runs Are Extremely Slow

```ini
[defaults]
forks = 20
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts_cache
fact_caching_timeout = 86400

[ssh_connection]
pipelining = True
```
Checklist: is `pipelining` enabled (biggest single win, cuts SSH round-trips significantly)? Is fact gathering happening unnecessarily on every run against every host (`gather_facts: false` where facts aren't actually needed, or enable fact caching)? Are `forks` set too low for your inventory size (default is only 5, often far too low for fleets of dozens/hundreds of hosts)?

---

### 7.2 A Specific Task Is Disproportionately Slow

```bash
ANSIBLE_CALLBACK_WHITELIST=profile_tasks ansible-playbook site.yml
```
Enable the `profile_tasks` callback plugin to get a per-task timing breakdown at the end of the run, pinpointing exactly which task(s) are consuming the most time rather than guessing.

---

### 7.3 High Memory Usage on the Control Node with Large Inventories

Reduce `forks` if memory-constrained (parallelism trades memory for speed), and consider splitting an extremely large inventory/playbook run into smaller batched runs (`--limit` against subsets) if a single run against thousands of hosts is overwhelming the control node's resources.

---

### 7.4 `pipelining = True` Not Actually Improving Performance

Confirm `requiretty` is disabled in `/etc/sudoers` on managed nodes — pipelining silently has no effect (or can cause failures) if `requiretty` is still set, since it interferes with the mechanism pipelining relies on.

```bash
# On managed nodes, comment out or ensure this isn't present in /etc/sudoers:
# Defaults requiretty
```

---

## MODULE 8: Ansible Vault Issues

### 8.1 "Decryption failed" Errors

```bash
ansible-vault view secrets.yml --vault-password-file=~/.vault_pass.txt
```
Confirm you're using the correct vault password/vault-id for that specific file (especially in multi-vault-ID setups where prod/dev secrets use different passwords), and that the password file itself doesn't have trailing whitespace/newline issues corrupting the password value.

---

### 8.2 Playbook Fails with "Attempting to decrypt but no vault secrets found"

```bash
ansible-playbook site.yml --ask-vault-pass
# or
ansible-playbook site.yml --vault-password-file=~/.vault_pass.txt
```
The playbook references vault-encrypted content but wasn't given any way to decrypt it at runtime — ensure `ansible.cfg` has a `vault_password_file` configured, or the command line explicitly supplies one.

```ini
[defaults]
vault_password_file = ~/.vault_pass.txt
```

---

### 8.3 Vault-Encrypted Variable Breaks YAML Structure

```yaml
# Correct format for an individually-encrypted variable
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          663864396235633965663963343163336332...
```
Common cause: incorrect indentation of the encrypted block (it must be indented consistently under the `!vault |` block scalar), or accidentally re-encrypting an already-encrypted value, producing nested/corrupted vault data.

---

### 8.4 Rotating a Vault Password Across Many Encrypted Files

```bash
ansible-vault rekey secrets.yml --new-vault-password-file=new_pass.txt
find . -name "*.yml" -exec ansible-vault rekey {} --new-vault-password-file=new_pass.txt \;
```
Plan vault password rotation carefully across a whole repository — a partially-completed rekey operation across many files can leave some encrypted with the old password and some with the new, causing confusing intermittent decryption failures until fully reconciled.

---

## MODULE 9: Dynamic Inventory & Cloud Issues

### 9.1 Dynamic Inventory Plugin Returns Empty / No Hosts

```bash
ansible-inventory -i inventory/aws_ec2.yml --list --vvv
```
Checklist: confirm cloud credentials are actually available in the environment the inventory plugin runs in (AWS credentials via env vars/profile, GCP application-default credentials, etc.), check `filters`/`regions` in the inventory config aren't excluding the instances you expect, and confirm the relevant collection (`amazon.aws`, `google.cloud`, `azure.azcollection`) is actually installed.

---

### 9.2 Dynamic Inventory Is Extremely Slow

```yaml
# inventory/aws_ec2.yml
cache: true
cache_plugin: jsonfile
cache_timeout: 300
cache_connection: /tmp/aws_inventory_cache
```
Enable inventory caching so Ansible doesn't re-query the cloud API's full instance list on every single command/playbook run — particularly important for large fleets or CI pipelines running frequently.

---

### 9.3 New Cloud Instances Not Appearing in Inventory Immediately

Cloud provider eventual consistency (a newly launched instance's tags/metadata may take a few seconds to fully propagate) combined with inventory caching (see above) can cause a brand-new instance to be temporarily invisible — either disable caching for time-sensitive provisioning workflows, or add a short wait/retry loop after provisioning before expecting the instance to appear in a subsequent inventory refresh.

---

### 9.4 `keyed_groups` Not Creating Expected Groups

```bash
ansible-inventory -i inventory/aws_ec2.yml --graph
```
Confirm the tag/attribute key referenced in `keyed_groups` actually exists and is spelled correctly on the cloud resources (case-sensitive), and that the `prefix` doesn't collide with an existing static group name causing unexpected merging behavior.

---

## MODULE 10: Role & Collection Issues

### 10.1 Role Variables Not Being Applied as Expected

```bash
ansible-playbook site.yml -e "@vars_debug.yml" --extra-vars "ansible_check_mode=true"
```
Recall role `defaults/main.yml` is the **lowest** precedence — if a value isn't behaving as expected, check whether it's being unintentionally overridden by group_vars/host_vars/extra-vars at a higher precedence layer, using `ansible-inventory --host <hostname>` to see the final resolved value.

---

### 10.2 "ERROR! Unable to retrieve file contents" When Installing a Galaxy Role/Collection

```bash
ansible-galaxy install geerlingguy.nginx -vvv
```
Common causes: network/proxy issues reaching Galaxy's servers, a role/collection version pinned in `requirements.yml` that no longer exists (was removed/renamed upstream), or corporate firewall blocking `galaxy.ansible.com` — consider hosting an internal Galaxy-compatible mirror for air-gapped/restricted environments.

---

### 10.3 Collection Version Conflicts Between Roles

```bash
ansible-galaxy collection list
```
Different roles/playbooks in the same project may implicitly assume different collection versions — pin exact versions in a single, shared `requirements.yml` for the whole project rather than letting each role's own dependency resolution potentially install conflicting versions.

---

### 10.4 Role Dependency Loop or Unexpected Execution Order

```bash
ansible-playbook site.yml --list-tasks
```
Use `--list-tasks` to see the actual resolved execution order across role dependencies (`meta/main.yml`) before running — role dependencies execute before the role's own tasks, and circular dependencies between roles will cause Ansible to error out rather than infinite-loop, but the resulting error message can be non-obvious without tracing the dependency chain manually.

---

## MODULE 11: AWX / Tower Issues

### 11.1 Job Template Fails But the Same Playbook Works Fine from the CLI

```
Check the Job Template's associated Credentials, Inventory source, and Execution Environment —
these are often the actual difference, not the playbook logic itself.
```
Common causes: the AWX execution environment's bundled Ansible/collection versions differ from what's installed on your local CLI, credentials configured in AWX don't match what you're using locally (e.g., a different SSH key), or the Project hasn't synced the latest playbook changes from Git yet.

---

### 11.2 Project Sync Fails ("Failed to update project")

Check AWX's ability to reach the Git repository (network/firewall from the AWX execution node, not your local machine), confirm the Git credential configured in AWX is valid and has the correct access scope, and check for a webhook/branch mismatch if the Project is configured to track a specific branch that no longer exists.

---

### 11.3 Job Hangs in "Running" State Indefinitely

Check the AWX task/execution node's own resource utilization (CPU/memory exhaustion on the AWX infrastructure itself can cause jobs to appear stuck), review the job's live output for a task that's genuinely waiting on something (a prompt, an unreachable host with a long timeout), and check for AWX task queue backlog if many jobs are queued simultaneously against limited execution capacity.

---

### 11.4 RBAC — A User Can See a Job Template but Can't Launch It

Confirm the user/team has the "Execute" permission specifically (not just "Read") on that Job Template — AWX's RBAC model distinguishes between viewing, editing, and launching permissions separately, and it's a common setup mistake to grant only read access when execute access was intended.

---

## MODULE 12: CI/CD Integration Issues

### 12.1 Ansible Playbook Works Locally but Fails in CI Pipeline

```bash
# In the CI job, add:
ansible --version
ansible-galaxy collection list
python3 --version
```
Common causes: the CI runner has a different Ansible/Python version or missing collections compared to your local machine, SSH keys/credentials not properly injected as CI secrets, or the CI runner's network can't reach the target hosts (different network context than your local machine/VPN).

---

### 12.2 Vault Password Not Available in CI

```yaml
# GitHub Actions example
- name: Run Ansible playbook
  run: ansible-playbook site.yml --vault-password-file=<(echo "$VAULT_PASSWORD")
  env:
    VAULT_PASSWORD: ${{ secrets.ANSIBLE_VAULT_PASSWORD }}
```
Never hardcode the vault password in the pipeline YAML — inject it via the CI platform's native secrets store, and pass it as a process substitution/temporary file rather than writing it to a persistent file on the CI runner's disk.

---

### 12.3 CI Pipeline SSH Known-Hosts Verification Failing for Ephemeral Runners

```yaml
- name: Add hosts to known_hosts
  run: ssh-keyscan -H ${{ matrix.host }} >> ~/.ssh/known_hosts
```
Since CI runners are typically ephemeral (fresh environment every run), pre-populate `known_hosts` explicitly as a pipeline step rather than disabling host key checking entirely, preserving a meaningful security control even in a repeatable/automated context.

---

## Quick Reference: Most Useful Ansible Troubleshooting Commands

```bash
# Connectivity
ansible <host> -m ping -vvv
ssh -v user@host

# Syntax & Preview
ansible-playbook site.yml --syntax-check
ansible-playbook site.yml --list-tasks --list-hosts

# Deep Debugging
ansible-playbook site.yml -vvv
ansible-playbook site.yml --step
ansible-playbook site.yml --start-at-task="Task name"

# Variables
ansible-inventory -i inventory/hosts.ini --host <hostname>
ansible <host> -m debug -a "var=hostvars[inventory_hostname]"

# Vault
ansible-vault view secrets.yml --vault-password-file=~/.vault_pass.txt

# Performance
ANSIBLE_CALLBACK_WHITELIST=profile_tasks ansible-playbook site.yml

# Dynamic Inventory
ansible-inventory -i inventory/aws_ec2.yml --list --vvv
ansible-inventory -i inventory/aws_ec2.yml --graph
```

---
