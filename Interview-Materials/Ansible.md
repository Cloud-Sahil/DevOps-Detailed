# Ansible Interview Questions — Simple + Scenario-Based (155+ Questions)

---

## Table of Contents

1. [Ansible Fundamentals (Simple)](#module-1-ansible-fundamentals-simple) — 15
2. [Inventory Management](#module-2-inventory-management) — 12
3. [Playbooks & YAML](#module-3-playbooks--yaml) — 14
4. [Variables & Facts](#module-4-variables--facts) — 14
5. [Modules & Idempotency](#module-5-modules--idempotency) — 14
6. [Conditionals, Loops & Handlers](#module-6-conditionals-loops--handlers) — 12
7. [Roles & Ansible Galaxy](#module-7-roles--ansible-galaxy) — 10
8. [Templates (Jinja2)](#module-8-templates-jinja2) — 8
9. [Ansible Vault](#module-9-ansible-vault) — 10
10. [Error Handling & Debugging](#module-10-error-handling--debugging) — 10
11. [Cloud Integration & Dynamic Inventory](#module-11-cloud-integration--dynamic-inventory) — 10
12. [AWX / Ansible Automation Platform](#module-12-awx--ansible-automation-platform) — 8
13. [Performance & Best Practices](#module-13-performance--best-practices) — 8
14. [Advanced Real-World Scenarios](#module-14-advanced-real-world-scenarios) — 20

**Total: 165 Questions**

---

## MODULE 1: Ansible Fundamentals (Simple)

**1. What is Ansible?**
An open-source, agentless automation tool for configuration management, application deployment, and orchestration, using SSH/WinRM to communicate with managed nodes.

**2. Why is Ansible called "agentless"?**
Because it doesn't require any persistent daemon/agent installed on managed nodes — it connects via SSH (or WinRM for Windows) and only needs Python present on the target for most modules.

**3. What is a Control Node?**
The machine where Ansible is installed and from which playbooks/commands are executed.

**4. What is a Managed Node?**
A target server that Ansible connects to and configures — requires no agent, just SSH access and (usually) Python.

**5. What is an Inventory in Ansible?**
The list of managed hosts (and groups of hosts) that Ansible operates against, defined in INI, YAML, or dynamically via a cloud plugin.

**6. What is a Playbook?**
A YAML file defining an ordered set of tasks (organized into "plays") to be executed against a set of hosts.

**7. What is a Module in Ansible?**
A discrete unit of work Ansible executes on a managed node (e.g., `apt`, `copy`, `service`) — the actual "verbs" of automation.

**8. What is the difference between push and pull automation models, and which does Ansible use?**
Push (Ansible) sends/executes changes on demand from the control node; pull (e.g., Puppet/Chef agents) has agents periodically fetch and apply configuration independently.

**9. What is `ansible.cfg`?**
Ansible's configuration file controlling defaults like inventory path, remote user, privilege escalation settings, and performance tuning.

**10. What is the default number of parallel forks in Ansible, and why does it matter?**
5 — often too low for larger fleets; increasing `forks` in `ansible.cfg` improves execution parallelism/speed.

**11. What's the difference between Ansible and Terraform?**
Terraform is primarily for provisioning infrastructure (declarative resource creation); Ansible is primarily for configuration management and application deployment on existing infrastructure — they're often used together (Terraform provisions, Ansible configures).

**12. What is idempotency, and why does it matter in Ansible?**
Running the same operation multiple times produces the same end state without unintended side effects — critical because it lets you safely re-run playbooks repeatedly without worrying about duplicating actions or causing drift.

**13. What is a "play" versus a "playbook"?**
A play is a single mapping of hosts to tasks within a playbook; a playbook is the overall YAML file, which can contain multiple plays targeting different host groups.

**14. What is `become` used for in Ansible?**
Privilege escalation (like `sudo`) — running a task as a different user (typically root) than the one used to establish the SSH connection.

**15. What does the `-i` flag do in `ansible-playbook`?**
Specifies the inventory file/source to use for that run, overriding the default configured in `ansible.cfg`.

---

## MODULE 2: Inventory Management

**16. What are the two main inventory formats Ansible supports?**
INI format and YAML format — both can define hosts, groups, and group-of-groups (`children`).

**17. What's the difference between `group_vars` and `host_vars`?**
`group_vars` define variables applied to all hosts in a group; `host_vars` define variables specific to a single host — both are auto-loaded by Ansible based on directory naming convention.

**18. Scenario: You need a variable applied to all hosts except one specific host in a group. How do you handle this cleanly?**
Define the variable at the group level in `group_vars/<group>.yml`, then override it specifically in `host_vars/<specific_host>.yml` — since host_vars take precedence over group_vars.

**19. What is a "children" group in Ansible inventory?**
A group that contains other groups (rather than individual hosts directly), allowing hierarchical grouping (e.g., a `production` group containing both `webservers` and `dbservers` as children).

**20. What is dynamic inventory, and why is it needed for cloud environments?**
Inventory generated in real-time by querying a cloud provider's API (rather than a static file), necessary because cloud instances are often ephemeral/frequently changing, making static inventory unmanageable.

**21. Scenario: How would you target only hosts that are in both the `webservers` group AND the `production` group?**
Use the intersection pattern: `ansible "webservers:&production" -m ping`.

**22. What does the `ansible_host` variable do?**
Overrides the actual IP/hostname Ansible connects to for a given inventory hostname — useful when the inventory name (an alias) differs from the real connection address.

**23. Scenario: You want to exclude a specific host from a playbook run without editing the inventory file. How?**
Use `--limit "all:!hostname"` on the command line, or a pattern like `ansible-playbook site.yml --limit "webservers:!web2.example.com"`.

**24. What is `ansible_connection`, and what values can it take?**
Specifies the connection plugin used to reach a host — common values include `ssh` (default), `local` (for localhost tasks), `winrm` (Windows), and `docker` (containers).

**25. Scenario: How would you run a task only on the control node itself, not on any remote managed hosts?**
Target the `localhost` group with `ansible_connection: local` (often implicit for `localhost`), e.g., `hosts: localhost` combined with `connection: local` in the play.

**26. What is the purpose of the `[group:vars]` syntax in INI inventory?**
Defines variables that apply to all hosts within that specific group directly inside the inventory file (an alternative to a separate `group_vars/` file).

**27. Scenario: Your organization manages thousands of ephemeral cloud instances. What inventory approach would you recommend and why?**
Dynamic inventory using the appropriate cloud plugin (e.g., `amazon.aws.aws_ec2`), combined with inventory caching to avoid excessive API calls on every run — static inventory would be unmanageable at that scale and staleness-prone.

---

## MODULE 3: Playbooks & YAML

**28. What is the correct YAML document start marker, and is it mandatory?**
`---` at the top of the file — conventional but not strictly mandatory for a single-document YAML file, though good practice to include.

**29. Scenario: A playbook fails immediately with a YAML parsing error, but the file "looks correct" visually. What's the most likely cause?**
Mixed tabs and spaces in indentation (YAML is whitespace-sensitive and doesn't tolerate mixed tab/space indentation), or an unquoted string starting with `{{ }}` being misinterpreted as a YAML flow mapping.

**30. What does `--check` mode do?**
Runs the playbook in "dry-run" mode, reporting what *would* change without actually applying any changes — useful for validating a playbook's intended impact before running it for real.

**31. What does `--diff` do, and when is it especially useful?**
Shows the actual content differences for tasks that modify files (like `template`/`copy`/`lineinfile`) — especially useful combined with `--check` to preview exact file changes before applying them.

**32. Scenario: How would you preview which hosts and tasks a playbook would execute, without actually running anything?**
`ansible-playbook site.yml --list-hosts` and `ansible-playbook site.yml --list-tasks`.

**33. What is the purpose of multiple plays within a single playbook file?**
To target different groups of hosts with different sets of tasks sequentially within one orchestrated run (e.g., configure web tier, then database tier).

**34. Scenario: A multi-tier deployment playbook needs to configure the database tier fully before starting on the web tier (which depends on DB connection details). How do you structure this?**
Define the database-tier play first in the playbook file — plays execute strictly in the order they appear, so ordering the DB play before the web play (potentially using `set_fact`/`add_host` to pass DB connection info forward) ensures the correct sequencing.

**35. What is `gather_facts`, and why might you disable it?**
Controls whether Ansible automatically collects system facts (OS, memory, network info) at the start of a play — disabling it (`gather_facts: false`) speeds up runs when facts aren't actually needed by any task.

**36. Scenario: A very large playbook run against hundreds of hosts is slow primarily due to fact gathering. What are two ways to address this?**
Disable fact gathering entirely if facts aren't used (`gather_facts: false`), or enable fact caching (`fact_caching = jsonfile` in `ansible.cfg`) so facts are reused across runs within a timeout window instead of being re-gathered every single time.

**37. What is the difference between `include_tasks` and `import_tasks`?**
`import_tasks` is processed statically at parse time (before the playbook runs, so conditionals on the import itself apply to all included tasks uniformly); `include_tasks` is processed dynamically at runtime (allowing more flexible, per-task conditional logic and variables that aren't known until execution time).

**38. Scenario: You need to conditionally include an entire set of tasks only for certain hosts, with the condition depending on a fact gathered at runtime. Which should you use — `include_tasks` or `import_tasks` — and why?**
`include_tasks`, since it's evaluated dynamically at runtime and can properly react to runtime-gathered facts/variables, whereas `import_tasks` is resolved statically before the play even begins executing.

**39. What is the `serial` keyword used for in a play?**
Controls how many hosts are processed at a time in a rolling fashion (e.g., `serial: 2` processes 2 hosts fully before moving to the next 2) — useful for rolling deployments where you don't want to update all hosts simultaneously.

**40. Scenario: How would you perform a rolling update across 20 web servers, updating only 25% at a time to avoid a full-fleet outage if something goes wrong?**
Use `serial: "25%"` on the play, so Ansible processes hosts in batches representing 25% of the total inventory at a time, allowing you to catch a failure early (combined with `max_fail_percentage`) before it affects the whole fleet.

**41. What does `strategy: free` do, differing from the default `linear` strategy?**
`linear` (default) waits for all hosts to complete each task before moving to the next task; `free` allows each host to proceed through the task list independently at its own pace, without waiting for slower hosts — useful when hosts have no interdependency on task timing.

---

## MODULE 4: Variables & Facts

**42. What is the highest-precedence variable source in Ansible?**
Extra vars passed via `-e` on the command line — they override every other variable source.

**43. What is the lowest-precedence variable source?**
Role `defaults/main.yml` — the easiest values to override, intended as sensible fallback defaults.

**44. Scenario: A variable defined in `group_vars/all.yml` isn't taking effect because a more specific value seems to be winning. How do you find out which layer is actually determining the final value?**
Use `ansible-inventory -i inventory/hosts.ini --host <hostname>` to see the fully resolved variable value for that host, and reason through the precedence order to identify the higher-precedence source overriding it.

**45. What are Ansible Facts?**
Auto-discovered system information (OS, network interfaces, memory, CPU) gathered from managed nodes at the start of a play (unless disabled), accessible via `ansible_facts['...']`.

**46. Scenario: You need faster playbook runs but some tasks still rely on OS-family facts. What's a good middle-ground solution instead of just disabling fact gathering entirely?**
Enable fact caching (`fact_caching = jsonfile`, with an appropriate `fact_caching_timeout`) so facts are gathered once and reused across subsequent runs within the cache window, rather than re-gathering from scratch every single execution.

**47. What does `register` do?**
Captures the output/result of a task into a variable, allowing subsequent tasks to reference that result (e.g., for conditional logic based on a previous task's outcome).

**48. Scenario: How would you make a task only run if a specific file does NOT already exist, using `register`?**
```yaml
- stat: path=/etc/app/config.yml
  register: config_check
- copy: src=config.yml dest=/etc/app/config.yml
  when: not config_check.stat.exists
```

**49. What is `set_fact` used for?**
Defining a new variable dynamically during playbook execution, often computed from other variables or task results — useful for derived/calculated values.

**50. Scenario: You need a fact set on one host to be available and usable on a different host later in the same play (e.g., a database's IP needed by web servers). How do you achieve this?**
Use `set_fact` with `cacheable: true` combined with fact caching enabled, or more directly, use `hostvars['db_host']['some_fact']` syntax to reference another host's already-gathered facts/registered variables directly within the same play run.

**51. What is the difference between `vars`, `vars_files`, and `vars_prompt`?**
`vars` defines variables inline in the playbook; `vars_files` loads variables from external YAML files; `vars_prompt` interactively prompts the user for a variable value at runtime.

**52. Scenario: A deployment playbook needs to prompt for a version number interactively when run manually, but should also support being run non-interactively in CI with the version passed as an argument. How do you design this?**
Use `vars_prompt` for the interactive default behavior, but since `-e` (extra-vars) always takes precedence over other variable sources, CI can simply pass `-e "version=1.2.3"` to bypass the prompt entirely without needing separate playbook logic.

**53. What is `hostvars`, and when would you use it?**
A special variable giving access to variables/facts belonging to *other* hosts in the inventory (not just the current host being processed) — commonly used when one host's configuration depends on another host's IP/facts (e.g., web servers needing the database server's address).

**54. Scenario: How would you build a load balancer configuration file that lists the IP addresses of all currently healthy web servers in the `webservers` group?**
```jinja2
{% for host in groups['webservers'] %}
server {{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }};
{% endfor %}
```
Using the `groups['webservers']` list combined with `hostvars` to pull each host's facts.

**55. What does the `default()` Jinja2 filter do, and why is it a best practice to use liberally?**
Provides a fallback value if a variable is undefined, preventing "undefined variable" errors — a defensive best practice especially in templates/roles meant to be reused across environments where not every variable is guaranteed to be set.

---

## MODULE 5: Modules & Idempotency

**56. What's the difference between the `command` and `shell` modules?**
`command` runs a command directly without shell processing (no pipes, redirects, or environment variable expansion) and is safer/preferred by default; `shell` runs through `/bin/sh`, supporting shell features like pipes — use only when genuinely necessary.

**57. Why is the `raw` module different from `command`/`shell`, and when would you use it?**
`raw` bypasses the Python module system entirely, sending the command directly over SSH — used for bootstrapping hosts that don't yet have Python installed (a prerequisite for most other modules to function).

**58. Scenario: A `command` task is reported as "changed" on every single playbook run, even though it doesn't actually modify anything meaningful. How do you fix the reporting?**
Add `changed_when: false` (if it truly never changes state) or use `creates`/`removes` arguments to make the task properly conditional and idempotent.

**59. Why are `command` and `shell` considered "not idempotent by default"?**
Because they simply execute the given command every time regardless of current system state, unlike purpose-built modules (like `apt` or `service`) which check current state first and only act if a change is actually needed.

**60. What does the `creates` argument do on a `command`/`shell` task?**
Skips running the task if the specified file/path already exists — a simple way to make an otherwise non-idempotent command idempotent.

**61. Scenario: How would you install a specific version of a package (not just "latest" or "present") using the `apt` module?**
```yaml
apt:
  name: nginx=1.18.0-0ubuntu1
  state: present
```

**62. What's the difference between `state: present` and `state: latest` in package management modules?**
`present` ensures the package is installed at any version (won't upgrade an already-installed version); `latest` ensures the package is upgraded to the newest available version every run — `latest` can cause unexpected drift/changes on every run if the repository has newer versions available.

**63. Scenario: You need to ensure a config file line is present without disturbing the rest of the file's content. Which module fits, and how?**
`lineinfile` — it inserts, modifies, or ensures a specific line matching a regex is present, without needing to manage the entire file's content like `copy`/`template` would.

**64. What is the risk of using `state: latest` broadly across a fleet, and how do you mitigate it?**
It can cause unplanned, inconsistent package version drift across a fleet (and unexpected "changed" results/restarts) as upstream repositories update — mitigate by pinning specific versions explicitly for production-critical packages instead of relying on "latest."

**65. Scenario: A `template` task keeps failing with a permission error, but a `copy` task to the same destination path works fine. What's the likely cause?**
The template rendering itself might reference an undefined variable causing a Jinja2 error being misreported, or (more likely for a genuine permission issue) the destination directory ownership/mode differs from what's expected — check with `--diff -vvv` for the actual underlying error rather than assuming it's purely a permissions issue based on the message alone.

**66. What does the `notify` keyword do on a task?**
Triggers a named handler to run at the end of the play, but only if the task actually reports "changed" — and only once even if multiple tasks notify the same handler.

**67. Scenario: How would you ensure a specific task always executes, completely bypassing any `when` conditions or normal idempotency checks, purely for diagnostic purposes?**
Add a separate diagnostic task without a `when` condition and with `changed_when: false` if it's read-only, run explicitly perhaps gated behind a tag (`--tags diagnostics`) so it's opt-in rather than part of normal runs.

**68. What is the `assert` module used for?**
Validates that a given condition is true, failing the playbook with a custom error message if not — useful for pre-flight checks (e.g., "confirm the required variable is set and non-empty before proceeding").

**69. Scenario: You want a playbook to fail immediately and clearly if a required variable wasn't passed in via `-e`, rather than failing later with a confusing "undefined variable" error deep in the task list.**
```yaml
- name: Validate required variable
  assert:
    that:
      - app_version is defined
      - app_version | length > 0
    fail_msg: "app_version must be provided via -e"
```

---

## MODULE 6: Conditionals, Loops & Handlers

**70. What does the `when` keyword do?**
Conditionally executes a task only if the specified expression evaluates to true.

**71. Scenario: How would you run a task only on RedHat-family hosts?**
```yaml
when: ansible_facts['os_family'] == "RedHat"
```

**72. What's the difference between listing multiple conditions in a `when` list versus using `and`/`or` explicitly?**
A YAML list under `when:` is implicitly ANDed together (all conditions must be true); explicit `and`/`or` keywords are needed within a single string expression to combine conditions with OR logic or more complex boolean combinations.

**73. What does `loop` do, and how is it different from the older `with_items`?**
`loop` iterates a task over a list of items (the modern, recommended syntax); `with_items` is the older syntax being phased out in favor of `loop` combined with appropriate lookup plugins for more complex iteration needs.

**74. Scenario: How would you loop over a list of dictionaries to create multiple users with different group memberships?**
```yaml
- user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
  loop:
    - { name: 'alice', groups: 'admin' }
    - { name: 'bob', groups: 'dev' }
```

**75. What is a handler, and how does it differ from a regular task?**
A handler is a task that only runs when explicitly notified by another task (via `notify`), and runs at most once at the end of the play regardless of how many times it was notified — used for actions like service restarts that should happen once after all relevant changes are made.

**76. Scenario: You have three separate tasks that each modify a different nginx config file, and you want nginx restarted only once at the end if any of them changed anything. How do you structure this?**
Have each of the three tasks `notify: Restart nginx`, and define a single `Restart nginx` handler — Ansible automatically deduplicates and runs it only once at the end of the play, even if all three tasks reported "changed."

**77. What does `meta: flush_handlers` do, and when would you use it?**
Forces any pending notified handlers to run immediately, rather than waiting until the natural end of the play — useful when a later task in the same play genuinely depends on the handler's action (e.g., a service restart) having already completed.

**78. What is a `block`, and what problem does it solve?**
Groups multiple tasks together to apply a shared `when` condition, `become` setting, or tags to all of them at once, and enables `rescue`/`always` error-handling semantics similar to try/catch/finally.

**79. Scenario: How would you implement "try this deployment step, and if it fails, automatically roll back, but always send a notification regardless of outcome"?**
```yaml
- block:
    - command: /opt/deploy.sh
  rescue:
    - command: /opt/rollback.sh
  always:
    - debug: msg="Deployment attempt complete"
```

**80. What does `ignore_errors: true` do, and what's a risk of overusing it?**
Allows the playbook to continue even if that specific task fails; overusing it can mask genuine failures that should have stopped the run, leading to a false sense of a "successful" deployment when something actually silently failed partway through.

**81. Scenario: A task's failure should be tolerated only under specific conditions (e.g., "it's fine if this fails because the resource doesn't exist yet"), but should still fail loudly for any other unexpected error. How do you handle this more precisely than blanket `ignore_errors`?**
Use `failed_when` combined with `register` to define exactly what output/condition constitutes an acceptable vs. unacceptable failure, rather than ignoring all errors indiscriminately regardless of cause.

---

## MODULE 7: Roles & Ansible Galaxy

**82. What is a role in Ansible?**
A structured, reusable collection of tasks, variables, handlers, templates, and files organized in a standard directory layout, allowing configuration logic to be packaged and reused across multiple playbooks/projects.

**83. What's the difference between `roles/nginx/defaults/main.yml` and `roles/nginx/vars/main.yml`?**
`defaults` holds the lowest-precedence variables meant to be easily overridden by consumers of the role; `vars` holds higher-precedence variables generally not meant to be casually overridden by every consumer.

**84. Scenario: You're building a reusable "nginx" role to be shared across multiple teams/projects. Where should the default HTTP port value live, and why?**
In `defaults/main.yml` — since it's a value teams will very likely want to override per their own environment's needs, and defaults are specifically designed to be easily overridden without needing to fork or modify the role itself.

**85. What does `ansible-galaxy init` do?**
Scaffolds the standard directory structure for a new role (or collection), saving time versus manually creating each expected subdirectory.

**86. What is `meta/main.yml` used for in a role?**
Declares role metadata and dependencies — other roles that should automatically run before this role's own tasks.

**87. Scenario: Your "app" role requires a "common" hardening role and a "firewall" role to always run first, in that order, whenever "app" is used. How do you enforce this without relying on playbook authors remembering to list them manually?**
Declare them as dependencies in the "app" role's `meta/main.yml`, so they automatically execute in the specified order before "app"'s own tasks run, regardless of what the consuming playbook explicitly lists.

**88. What is `requirements.yml` used for?**
Declares external roles/collections (and their pinned versions) a project depends on, installed via `ansible-galaxy install -r requirements.yml` — enabling reproducible environments across different machines/CI runners.

**89. Scenario: A colleague says "it worked on my machine" for a playbook using a community role, but it fails in CI with different behavior. What's a likely root cause and how do you prevent this going forward?**
An unpinned role/collection version resolving to different actual versions between the colleague's machine and the CI environment (installed at different times) — pin exact versions in `requirements.yml` so everyone (and CI) installs the identical, reproducible version.

**90. What's the difference between `roles:` in a playbook and `include_role`/`import_role`?**
The `roles:` keyword applies roles at the play level with a fixed, simple syntax; `include_role`/`import_role` are used within the `tasks:` section for more granular control, such as conditionally including a role or including it multiple times with different variables in the same play.

**91. Scenario: You need to apply the same role twice within a single play, but with different variables each time (e.g., configuring two different virtual hosts using the same nginx-vhost role). How?**
Use `include_role` within the tasks section (rather than the `roles:` keyword, which doesn't cleanly support applying the same role multiple times with different variables in one play) — calling it twice with different `vars` each time.

---

## MODULE 8: Templates (Jinja2)

**92. What's the key functional difference between the `copy` and `template` modules?**
`copy` transfers a file as-is (no variable substitution); `template` renders the source file as a Jinja2 template first, substituting variables/expressions, before placing the result at the destination.

**93. Scenario: A config file deployed via the `copy` module isn't having its `{{ variable }}` placeholders replaced. Why, and how do you fix it?**
`copy` doesn't perform Jinja2 rendering at all — switch to the `template` module, which does render Jinja2 expressions in the source file.

**94. What is a Jinja2 filter, and give an example?**
A function applied to a variable using the pipe (`|`) syntax to transform its value, e.g., `{{ my_list | join(", ") }}` or `{{ my_string | upper }}`.

**95. Scenario: How would you safely reference a variable in a template that might not always be defined, providing a sensible fallback?**
`{{ my_variable | default('fallback_value') }}` — using the `default` filter to avoid an undefined-variable error.

**96. How do you use conditionals inside a Jinja2 template file?**
```jinja2
{% if enable_ssl %}
listen 443 ssl;
{% else %}
listen 80;
{% endif %}
```

**97. Scenario: You need to generate an nginx upstream block listing every host in the `webservers` group with its IP address. How would you write this in a Jinja2 template?**
```jinja2
upstream backend {
{% for host in groups['webservers'] %}
    server {{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }};
{% endfor %}
}
```

**98. What does the `to_json`/`to_yaml` filter do, and when is it useful?**
Converts a data structure (dict/list) into a JSON/YAML string representation within a template — useful for generating structured config files (like a JSON application config) directly from Ansible variables.

**99. Scenario: A template renders correctly most of the time, but occasionally produces a diff/change on every single run even though the underlying data hasn't changed. What's a likely cause?**
Non-deterministic ordering of a dictionary/list being rendered inconsistently between runs (or a timestamp/random value embedded in the template), causing the rendered output to differ slightly each time even without an actual meaningful configuration change.

---

## MODULE 9: Ansible Vault

**100. What is Ansible Vault used for?**
Encrypting sensitive data (passwords, API keys, certificates) within Ansible files so they can be safely stored in version control without exposing plaintext secrets.

**101. What's the difference between `ansible-vault encrypt` and `ansible-vault encrypt_string`?**
`encrypt` encrypts an entire file; `encrypt_string` encrypts a single variable's value, allowing the surrounding YAML file to remain readable/diffable while only the specific sensitive value is encrypted.

**102. Scenario: Your team wants variable files to remain easily readable and diffable in pull requests, but specific sensitive values (like passwords) must still be protected. What Vault approach fits best?**
Use `ansible-vault encrypt_string` to encrypt only the specific sensitive variables individually, embedding them as `!vault` blocks within an otherwise plaintext, readable YAML file.

**103. What is a Vault ID, and why would you use multiple Vault IDs in one project?**
A labeled vault password allowing different encryption passwords for different purposes/environments (e.g., separate `dev` and `prod` vault passwords) — so a leaked development vault password doesn't compromise production secrets encrypted under a different vault ID.

**104. Scenario: How would you run a playbook that references both dev-vault-encrypted and prod-vault-encrypted variables in the same run?**
```bash
ansible-playbook site.yml --vault-id dev@~/.vault_pass_dev.txt --vault-id prod@~/.vault_pass_prod.txt
```

**105. What does `ansible-vault rekey` do?**
Changes the password used to encrypt a vault-encrypted file, re-encrypting its content under the new password.

**106. Scenario: How would you rotate the vault password across an entire repository of many encrypted files safely?**
Use `ansible-vault rekey` scripted across all relevant files (e.g., via `find ... -exec ansible-vault rekey`), being careful to complete the rotation atomically/quickly since a partially-completed rotation leaves some files under the old password and some under the new, causing confusing intermittent failures.

**107. Why shouldn't you commit the Vault password itself to the same Git repository as the encrypted secrets?**
Doing so defeats the entire purpose of encryption — anyone with repository access would have both the encrypted data and the key to decrypt it, providing no actual security benefit over storing the secrets in plaintext.

**108. Scenario: How would you integrate Ansible with an external secrets manager (like HashiCorp Vault or AWS Secrets Manager) instead of relying solely on Ansible Vault's built-in encryption?**
Use the relevant lookup plugin (`community.hashi_vault.hashi_vault` or `amazon.aws.aws_secret`) to fetch secrets at runtime directly from the external secrets manager, avoiding the need to store even encrypted secret values in the Ansible repository at all.

**109. What happens if you try to run a playbook referencing vault-encrypted content without providing a vault password?**
Ansible fails with an error indicating it needs a vault password/secret to decrypt the referenced content — you must supply `--ask-vault-pass` or `--vault-password-file` (or have `vault_password_file` configured in `ansible.cfg`).

---

## MODULE 10: Error Handling & Debugging

**110. What does the `-vvv` flag do?**
Increases verbosity of `ansible`/`ansible-playbook` output, showing detailed module input/output and connection debugging information — essential for diagnosing failures.

**111. Scenario: A playbook fails on one specific task deep in a long playbook, and you don't want to re-run everything from the beginning while debugging. What option helps?**
`--start-at-task="Task name"` — resumes execution from a specific named task rather than the beginning of the playbook.

**112. What is the purpose of `--step`?**
Prompts for confirmation before executing each individual task, letting you interactively step through a playbook to isolate exactly where an issue occurs.

**113. Scenario: You need to see exactly what tasks and hosts a playbook would touch before actually running it, especially before a production change. What two flags help?**
`--list-tasks` and `--list-hosts`.

**114. What does `failed_when` allow you to do that a module's default failure behavior doesn't?**
Define a custom condition for what constitutes "failure" for a task, based on registered output — useful when a command's default exit code/behavior doesn't align with what actually represents success/failure for your specific use case.

**115. Scenario: A `command` task should be considered "failed" if a specific string appears in its output, even though the command itself exits with code 0. How do you implement this?**
```yaml
- command: /opt/healthcheck.sh
  register: result
  failed_when: "'ERROR' in result.stdout"
```

**116. What does `any_errors_fatal: true` do at the play level?**
Immediately stops the entire playbook run across all hosts the moment any single host fails a task, rather than the default behavior of continuing against remaining hosts.

**117. Scenario: For a critical production deployment, you want the entire rollout to stop immediately if more than 20% of hosts fail, rather than continuing to roll out to a doomed fleet. How do you configure this?**
```yaml
- hosts: webservers
  max_fail_percentage: 20
```

**118. What is `ansible-lint`, and why is it valuable in a CI pipeline?**
A static analysis tool for Ansible content, catching style issues, deprecated syntax, and common mistakes before they cause runtime problems — valuable as an automated pre-merge/pre-deploy quality gate.

**119. What is Molecule, and what problem does it solve for testing Ansible roles?**
A testing framework that spins up ephemeral test environments (Docker/cloud) to verify a role behaves correctly and idempotently, rather than only manually testing roles against real production-adjacent infrastructure.

---

## MODULE 11: Cloud Integration & Dynamic Inventory

**120. Why is dynamic inventory preferred over static inventory files in cloud environments?**
Cloud instances are often ephemeral (Auto Scaling Groups, frequently recreated) — dynamic inventory queries the cloud provider's API in real time, always reflecting the current actual state, rather than requiring manual updates to a static file.

**121. What collection provides AWS dynamic inventory support?**
`amazon.aws` (specifically the `aws_ec2` inventory plugin).

**122. Scenario: How would you configure dynamic inventory to only include EC2 instances tagged `Environment: production`?**
```yaml
plugin: amazon.aws.aws_ec2
regions: [ap-south-1]
filters:
  tag:Environment: production
```

**123. What does `keyed_groups` do in a dynamic inventory configuration?**
Automatically creates inventory groups based on a resource attribute/tag value (e.g., grouping instances by their `Role` tag), avoiding manual group definitions for dynamically-varying attributes.

**124. Scenario: A newly provisioned cloud instance doesn't immediately show up when you query dynamic inventory right after creation. Why, and how do you address it?**
Likely due to cloud provider eventual consistency (tags/metadata not yet fully propagated) combined with inventory caching — either disable caching for time-sensitive provisioning workflows or add a short wait/retry before expecting the new instance to appear.

**125. What is the `add_host` module used for?**
Dynamically adds a host to the in-memory inventory during a playbook run — commonly used right after provisioning a new cloud instance so it can be immediately targeted by a subsequent play in the same playbook run.

**126. Scenario: You want to provision a new EC2 instance and then immediately configure it with a second play in the same playbook run, without it being in any pre-existing inventory. How?**
Use a cloud provisioning module (e.g., `amazon.aws.ec2_instance`) in the first play, register its result, then use `add_host` to add the new instance's IP to an in-memory group, which the second play then targets as its `hosts:`.

**127. How would you use Ansible together with Terraform in a typical infrastructure pipeline?**
Terraform provisions the underlying infrastructure (VMs, networking, security groups) and often outputs the resulting resource IPs/hostnames; Ansible then consumes those outputs (directly via a generated inventory file, or a dynamic inventory plugin) to handle OS-level configuration and application deployment on top of that provisioned infrastructure.

**128. What is inventory caching, and why is it particularly important for large-scale dynamic inventory setups?**
Caches the results of a dynamic inventory query for a configured timeout period, avoiding repeated, potentially slow/rate-limited API calls to the cloud provider on every single Ansible command — significantly improving performance for frequently-run playbooks/CI pipelines.

**129. Scenario: How would you configure Ansible to manage instances across three different cloud providers (AWS, GCP, Azure) in a single, unified inventory?**
Configure multiple dynamic inventory sources (one per cloud plugin: `aws_ec2.yml`, `gcp.yml`, `azure_rm.yml`) and pass them all together to Ansible via a directory-based inventory (`-i inventory/`) or a comma-separated list, letting Ansible merge them into a single unified inventory view.

---

## MODULE 12: AWX / Ansible Automation Platform

**130. What is AWX, and how does it relate to Red Hat Ansible Automation Platform (formerly Tower)?**
AWX is the open-source upstream project; Ansible Automation Platform is Red Hat's commercially supported, enterprise-hardened offering built from the same underlying codebase — analogous to the Kubernetes/OpenShift relationship.

**131. What problem does AWX solve that plain `ansible-playbook` from a laptop doesn't?**
Centralized, audited execution logging; secure credential storage (users can launch jobs without ever seeing the actual SSH keys/passwords); RBAC for controlling who can view/launch/edit specific automation; and scheduling/workflow orchestration across teams.

**132. What is a "Job Template" in AWX?**
A reusable, launchable bundle combining a specific playbook, inventory, credentials, and variables — the primary unit of "automation as a product" that end users interact with.

**133. Scenario: You want a junior engineer to be able to safely run a specific, well-tested playbook (e.g., "restart application service") without giving them direct SSH access or the ability to modify the underlying playbook logic. How does AWX help achieve this?**
Create a Job Template for that specific playbook with the appropriate credentials pre-configured, and grant the junior engineer RBAC "Execute" permission (not "Edit") on that Job Template — they can launch it through the UI/API without ever seeing the actual SSH credentials or being able to alter its logic.

**134. What is a "Workflow Template" in AWX, and how does it differ from a Job Template?**
A Workflow Template chains multiple Job Templates together (potentially with conditional branching based on success/failure), orchestrating a sequence of automation steps; a single Job Template represents just one playbook execution unit.

**135. What is an Execution Environment in AWX/Ansible Automation Platform?**
A containerized runtime image bundling a specific Ansible version, Python dependencies, and collections — ensuring consistent, reproducible playbook execution regardless of which underlying AWX node runs the job.

**136. Scenario: A playbook runs successfully via the CLI but fails when launched as a Job Template in AWX, with different behavior. What's the most likely explanation?**
The AWX Execution Environment likely bundles a different Ansible/collection version than your local CLI environment — check the EE's included versions against what you tested locally, since subtle version differences in modules/collections can cause behavioral discrepancies.

**137. What is a "Project" in AWX?**
A configured link to a source control repository (e.g., Git) containing the playbooks AWX will execute — AWX periodically syncs the Project to pull the latest playbook content before running jobs against it.

---

## MODULE 13: Performance & Best Practices

**138. What single configuration change often provides the biggest performance improvement for Ansible SSH-based execution?**
Enabling `pipelining = True` in the `[ssh_connection]` section of `ansible.cfg`, which reduces the number of SSH operations needed per task.

**139. Scenario: `pipelining = True` is enabled but doesn't seem to be improving performance at all. What's a commonly overlooked prerequisite?**
`requiretty` must be disabled in `/etc/sudoers` on the managed nodes — pipelining silently doesn't take effect (or can cause other issues) if `requiretty` is still active, since it interferes with the mechanism pipelining depends on.

**140. Why might increasing `forks` in `ansible.cfg` improve run time significantly for large inventories?**
The default of 5 forks means only 5 hosts are processed in parallel at a time — for an inventory of hundreds of hosts, this creates a serialized bottleneck; increasing forks allows far more hosts to be processed concurrently (bounded by control node resources).

**141. What is `ansible-lint`, and how would you integrate it into a CI/CD pipeline for Ansible content?**
A static analysis linter for Ansible playbooks/roles — integrate it as a required pre-merge CI check that fails the pipeline if style issues, deprecated syntax, or common anti-patterns are detected, catching problems before they reach production runs.

**142. Scenario: A team's roles work fine individually but behave inconsistently when combined in different playbooks across projects, due to differing collection versions being resolved. How do you prevent this?**
Maintain a single, shared `requirements.yml` pinning exact collection/role versions for the entire project/organization, rather than letting each role's own dependency resolution potentially install conflicting versions independently.

**143. What is the recommended module preference hierarchy — when should you use `shell`/`command` versus a purpose-built module?**
Always prefer a purpose-built module (`apt`, `copy`, `service`, `template`, etc.) when one exists, since these are properly idempotent and clearly express intent; use `command` (safer, no shell features) over `shell` (full shell processing, higher risk) only when no suitable dedicated module exists for the task.

**144. Scenario: How would you structure a large, multi-service Ansible codebase to avoid one giant, unmaintainable playbook file?**
Organize logic into well-scoped, reusable roles (one per logical component/service), use `group_vars`/`host_vars` to cleanly separate environment-specific configuration from logic, pin external dependencies via `requirements.yml`, and use tags to allow selective partial execution rather than needing entirely separate playbook files for every partial deployment scenario.

**145. What is the benefit of tagging tasks (`tags:`), and how would you use it in a real deployment scenario?**
Tags allow selective execution of a subset of tasks via `--tags`/`--skip-tags` without needing separate playbooks — e.g., tagging configuration tasks as `config` and deployment tasks as `deploy` lets you run only `--tags config` for a quick config-only update without triggering a full redeploy.

---

## MODULE 14: Advanced Real-World Scenarios

**146. Scenario: You need to perform a zero-downtime rolling deployment across 50 web servers behind a load balancer, removing each server from the LB before updating it and re-adding it only after a health check passes. How do you design this in Ansible?**
Use `serial` (e.g., `serial: 5` or a percentage) combined with explicit tasks in the play: deregister the host from the load balancer (via the relevant LB module, e.g., `amazon.aws.elb_target` or similar), perform the update, run a health check task with `until`/`retries` polling, then re-register the host with the load balancer — only proceeding to the next batch after the current batch's health checks pass.

**147. Scenario: A playbook run partway through a production deployment fails on host 15 of 50 due to a transient network blip, but hosts 1-14 were already successfully updated. How do you safely resume without re-running the whole thing or risking re-applying already-successful (and potentially non-idempotent) changes to hosts 1-14?**
Since well-designed Ansible tasks are idempotent, simply re-running the full playbook is often actually safe (hosts 1-14 would report "ok" not "changed" for already-applied changes) — but for genuinely non-idempotent steps, use the `--limit` flag combined with the `.ansible_retry` file Ansible automatically generates on failure (or a custom tracking mechanism) to resume specifically from the hosts that hadn't yet completed successfully.

**148. Scenario: How would you design an Ansible-based CI/CD pipeline for infrastructure configuration that requires mandatory human approval before applying changes to production, while allowing automatic application to lower environments?**
Structure the pipeline so `--check --diff` runs automatically on every change (posting the diff for review) for all environments; for production specifically, gate the actual `ansible-playbook` apply step behind a manual approval step in the CI/CD tool (e.g., a required reviewer/manual trigger in GitHub Actions/GitLab CI/Jenkins), while lower environments (dev/staging) proceed to auto-apply immediately after the check stage passes.

**149. Scenario: Your organization has 200 servers running a mix of Ubuntu, RHEL, and Windows, and you need one set of playbooks that correctly handles all three without three completely separate codebases. How do you architect this?**
Use OS-family-aware logic within shared roles — leverage the generic `package`/`service` modules (which delegate to the correct OS-specific backend automatically) combined with `ansible_facts['os_family']` conditionals for genuinely OS-specific naming/paths, and use `ansible_connection: winrm` targeting for Windows hosts within the same overall inventory/playbook structure rather than maintaining fully separate, duplicated playbook sets per OS.

**150. Scenario: A security audit requires proof that a specific hardening playbook has been successfully applied to 100% of your production fleet, with no configuration drift since. How would you provide this evidence using Ansible?**
Run the hardening playbook in `--check` mode on a scheduled basis (e.g., via AWX/cron) against the full fleet — since the playbook should be idempotent, a `--check` run reporting zero "changed" tasks across all hosts is direct, verifiable evidence of continued compliance; combine this with AWX's centralized job execution history/logs as the auditable record showing when these compliance checks ran and their results.

**151. Scenario: How would you handle a scenario where a critical security patch needs to be applied to production within the hour, bypassing your normal multi-day change-management review process, while still maintaining some safety guardrails?**
Design an expedited but still guarded pipeline path: a pre-approved, narrowly-scoped "emergency patch" playbook (already reviewed and tested in principle, limited to specific well-understood actions like a package update) that still requires at least one human approval click and runs with `serial`/health-check gating rather than a single big-bang blast to the entire fleet simultaneously, with a mandatory follow-up full review after the fact.

**152. Scenario: You need to migrate 500 servers from an old configuration management approach (manual scripts) to Ansible, without causing a disruptive flag-day cutover. What's a pragmatic migration strategy?**
Start by using Ansible in `--check` mode against the existing manually-configured servers to understand and document current-state drift versus your intended desired-state playbooks, gradually onboard servers in small batches (validating each batch thoroughly with `--check --diff` before the first real `apply`), and maintain the old manual process as a fallback only until each batch is confirmed stable under Ansible management — avoiding a single, risky full-fleet cutover.

**153. Scenario: A playbook that configures a clustered application (e.g., a database cluster) needs certain tasks to run on only the "primary" node and different tasks on "replica" nodes, with the primary determined dynamically based on current cluster state rather than a static inventory group. How do you design this?**
Add an early task (run against all cluster hosts) that queries the actual current cluster state (e.g., a database query or API call to determine which node is currently primary), register the result, and use `set_fact` combined with `when` conditions in later tasks to branch behavior based on the dynamically-determined role — rather than relying on a static inventory grouping that could become stale if the cluster fails over.

**154. Scenario: How would you design Ansible automation to safely rotate database credentials used by 30 different application servers, without any of them experiencing a connection failure during the rotation window?**
Prefer an architecture where credentials are fetched at runtime from a secrets manager (not baked into static config), so rotating the secret in the secrets manager (a separate, orchestrated step outside the app servers' knowledge) is transparent to the running applications — if static config files must be used, use a phased Ansible rollout that updates the credential in the database first to accept both old and new credentials temporarily, rolls out the new credential to all app servers via a rolling/serial deployment, then finally revokes the old credential only after confirming all app servers have successfully picked up the new one.

**155. Scenario: How would you use Ansible to enforce and continuously verify compliance with a security baseline (e.g., CIS benchmarks) across a large, heterogeneous fleet, and report on compliance drift over time?**
Use (or build on top of) an existing CIS-hardening role (many are available on Galaxy, e.g., community-maintained CIS roles), run it in `--check` mode on a recurring schedule via AWX, capture the results (which hosts report "changed" tasks, indicating drift from the baseline) into a centralized reporting system, and treat any host reporting unexpected changes during a `--check` compliance run as requiring investigation — since a change during a check run indicates the host's actual state has drifted from the enforced baseline since the last real apply.

**156. Scenario: Two different teams both have Ansible playbooks that occasionally run against overlapping infrastructure (e.g., a shared load balancer), and their concurrent runs have caused conflicting state changes. How do you prevent this architecturally?**
Establish clear ownership boundaries so only one team's automation is authorized to modify shared infrastructure components (enforced via AWX RBAC restricting who can launch Job Templates touching that specific inventory/infrastructure), and/or implement a locking mechanism (e.g., a coordination service or AWX's own concurrent job prevention settings on a Job Template) to serialize access to genuinely shared resources rather than allowing uncoordinated concurrent execution.

**157. Scenario: Your organization wants to start using Ansible for network device configuration (switches/routers) in addition to servers. What's fundamentally different about this compared to server automation, and how does Ansible accommodate it?**
Network devices typically don't run Python and can't use the standard SSH+Python module execution model — Ansible uses network-specific connection plugins (e.g., `ansible.netcommon.network_cli`) and modules (e.g., `cisco.ios.ios_config`) that communicate via the device's CLI/API directly rather than pushing and executing Python code on the device itself, requiring network-vendor-specific collections rather than generic OS modules.

**158. Scenario: How would you design a disaster recovery runbook using Ansible that can fully rebuild a critical application's server fleet from scratch in a new region, assuming the original region is completely unavailable?**
Ensure the application's entire configuration is captured declaratively in version-controlled playbooks/roles (not dependent on any manual, undocumented steps), combine with Terraform (or cloud-native provisioning) to stand up the new region's base infrastructure, then run the same Ansible playbooks against the newly provisioned DR infrastructure — treating "can we successfully run this playbook against a brand-new empty environment and get a fully working application" as the actual test of DR readiness, validated periodically via real game-day exercises rather than assumed to work.

**159. Scenario: A large enterprise wants standardized, self-service infrastructure configuration across dozens of teams, using Ansible, without each team needing deep Ansible expertise or direct server access. How would you design this platform?**
Build a curated library of well-tested, parameterized roles/playbooks covering common needs (e.g., "provision and configure a standard web server," "apply security baseline"), expose them as Job Templates in AWX/Ansible Automation Platform with sensible default surveys/prompts for the few parameters teams actually need to customize, and grant teams RBAC "Execute" access only — so teams get fast, safe self-service without needing to write or deeply understand the underlying Ansible code themselves.

**160. Scenario: How would you approach testing a complex Ansible role that manages a stateful service (like a database) to ensure it's genuinely idempotent and doesn't cause data loss on re-runs, before trusting it in production?**
Use Molecule to spin up an ephemeral test environment (Docker/cloud), run the role's `converge` step twice in sequence (the second run should report zero "changed" tasks if truly idempotent), and include Molecule verification steps that check actual data/state persistence assertions (not just "the role ran without error") to catch scenarios where a supposedly idempotent role might inadvertently reset or duplicate stateful data on a second run.

**161. Scenario: You need to enforce that no Ansible playbook can ever run with a `command`/`shell` module targeting a production database host without an explicit, reviewed exception, since these bypass normal idempotent safety guarantees. How would you implement this as an automated guardrail rather than a policy document?**
Integrate `ansible-lint` (potentially with a custom rule, or combined with a pre-commit/CI static analysis check) that scans playbooks for `command`/`shell` module usage specifically targeting groups/hosts tagged as production databases, failing the CI pipeline unless an explicit, documented exception annotation/tag is present — making the guardrail a technical, automatically-enforced pipeline gate rather than something relying purely on manual code review diligence.

**162. Scenario: How would you design Ansible automation to safely handle a scenario where a playbook needs to modify firewall rules on a host it's currently connected to via SSH, without risking locking itself out mid-run?**
Structure the firewall-rule-change tasks carefully with an explicit ordering that adds the new/correct rule *before* removing or restricting the old one (never a "flush all rules then add new ones" approach that could momentarily block the very SSH connection Ansible is using), and consider a safety-net task (e.g., a scheduled `at`/`cron` job that reverts the firewall change automatically after a few minutes unless explicitly confirmed) as a last-resort recovery mechanism if a rule change does inadvertently sever connectivity.

**163. Scenario: Your team wants to adopt GitOps-style practices for Ansible (similar to how Kubernetes uses ArgoCD), where the Git repository is the single source of truth and infrastructure state continuously reconciles to match it, rather than only running playbooks on manual trigger. How would you approach this with Ansible/AWX?**
Configure AWX Projects to auto-sync from Git on a schedule (or via webhook on merge), combined with Job Templates scheduled to run periodically (not just on-demand) so configuration drift is automatically detected and corrected on a recurring cadence, approximating a "continuous reconciliation" model — while being mindful that unlike Kubernetes' native declarative reconciliation loops, this is a periodic/scheduled approximation rather than true continuous real-time reconciliation, and communicating that distinction clearly to the team's expectations.

**164. Scenario: How would you handle secrets rotation for a fleet of 100 servers where each server needs a unique per-host secret (not a shared credential), using Ansible at scale?**
Use a `lookup` plugin integrated with a secrets manager that supports per-host secret paths (e.g., `lookup('amazon.aws.aws_secret', 'app/' + inventory_hostname + '/api_key')`), so each host's task dynamically fetches its own unique secret at runtime based on its inventory hostname, avoiding the need to manually manage or encrypt 100 individual per-host Vault-encrypted variables directly within the Ansible repository.

**165. Scenario: How would you future-proof your Ansible automation strategy for an organization increasingly adopting immutable infrastructure (e.g., building fully-baked machine images via Packer, rather than configuring long-lived servers in place)? What role does Ansible still play?**
Ansible remains valuable as the actual configuration logic executed *during* the image-building process (Packer commonly invokes Ansible as a provisioner to configure the image before it's baked), shifting Ansible's role from "continuously configuring long-lived servers in place" to "one-time configuration during immutable image creation" — while still using Ansible (or AWX) for any genuinely dynamic, runtime-only configuration (like registering with a service discovery system) that can't be baked into the image ahead of time.

---
