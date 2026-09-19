---
tags: [cloud, iac, ansible]
type: cheatsheet
source: Ansible CLI reference (compiled)
last-verified: 2026-08-25
---

# Ansible

## Up
- [[IaC]]

Agentless automation and configuration management. Ansible connects over SSH (WinRM on Windows) and applies declarative **playbooks** (YAML) to hosts defined in an **inventory** — no agent to install on targets.

## The command family
| Command | Purpose |
|---|---|
| `ansible` | Run a single ad-hoc module against hosts |
| `ansible-playbook` | Run a playbook |
| `ansible-inventory` | Inspect/validate inventory |
| `ansible-galaxy` | Install roles & collections |
| `ansible-vault` | Encrypt/decrypt secrets |
| `ansible-config` | View/dump configuration |
| `ansible-doc` | Module/plugin documentation |
| `ansible-pull` | Pull-mode (node pulls its own config from git) |
| `ansible-console` | Interactive REPL for ad-hoc commands |

## Ad-hoc commands
```bash
ansible all -m ping                       # connectivity check
ansible all -m command -a "uptime"        # run a command (default module)
ansible web -m shell -a "df -h | grep /"  # shell (pipes/redirs allowed)
ansible db -b -m apt -a "name=nginx state=present"   # -b = become root
ansible all -m copy -a "src=./f dest=/tmp/f mode=0644"
ansible all -m service -a "name=nginx state=restarted"
ansible all -m setup                      # gather facts
```
| Flag | Meaning |
|---|---|
| `-i <inventory>` | Inventory file/dir (or comma list `host1,host2,`) |
| `-m <module>` | Module to run (default `command`) |
| `-a "<args>"` | Module arguments |
| `-b` / `--become` | Privilege escalation (sudo) |
| `-K` / `--ask-become-pass` | Prompt for the sudo password |
| `-k` / `--ask-pass` | Prompt for the SSH password |
| `-u <user>` | Remote user |
| `-l <pattern>` | Limit to a subset of hosts |
| `-f <n>` | Parallelism (forks, default 5) |

## Inventory
**INI:**
```ini
[web]
web1.example.com
web2.example.com ansible_host=10.0.0.12

[db]
db1.example.com

[prod:children]
web
db

[web:vars]
ansible_user=ubuntu
```
Inspect it:
```bash
ansible-inventory -i inventory --list
ansible-inventory -i inventory --graph
ansible all -i inventory --list-hosts
```

## Running playbooks
```bash
ansible-playbook -i inventory site.yml
```
| Flag | Meaning |
|---|---|
| `--check` / `-C` | Dry run (no changes) |
| `--diff` | Show file/content differences |
| `--syntax-check` | Validate YAML/structure only |
| `--list-tasks` / `--list-hosts` / `--list-tags` | Preview without running |
| `-t <tags>` / `--tags` | Run only tagged tasks |
| `--skip-tags <tags>` | Skip tagged tasks |
| `-l <pattern>` / `--limit` | Limit hosts (`@retry` file works too) |
| `-e "k=v"` / `-e @vars.yml` | Extra variables (highest precedence) |
| `--start-at-task "<name>"` | Begin at a specific task |
| `--step` | Confirm each task interactively |
| `-K` | Ask for the become password |
| `-v` … `-vvvv` | Increasing verbosity |

## Playbook skeleton
```yaml
- name: Configure web servers
  hosts: web
  become: true
  vars:
    pkg: nginx
  tasks:
    - name: Install package
      ansible.builtin.apt:
        name: "{{ pkg }}"
        state: present
      notify: Restart nginx
  handlers:
    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

## Roles & collections (ansible-galaxy)
```bash
ansible-galaxy init myrole                       # scaffold a role
ansible-galaxy role install geerlingguy.nginx    # install a role
ansible-galaxy collection install community.docker
ansible-galaxy install -r requirements.yml       # install from requirements
ansible-galaxy collection list                   # list installed collections
```
`requirements.yml`:
```yaml
roles:
  - name: geerlingguy.postgresql
collections:
  - name: community.general
```

## Secrets (ansible-vault)
```bash
ansible-vault create secrets.yml       # new encrypted file
ansible-vault edit secrets.yml         # edit in place
ansible-vault view secrets.yml
ansible-vault encrypt vars.yml         # encrypt an existing file
ansible-vault decrypt vars.yml
ansible-vault rekey secrets.yml        # change the password
ansible-vault encrypt_string 's3cr3t' --name 'db_password'   # inline secret
# Use at runtime:
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass
```

## Config & docs
| Command | Description |
|---|---|
| `ansible-config dump --only-changed` | Show non-default settings |
| `ansible-config list` | All config options |
| `ansible-doc -l` | List modules |
| `ansible-doc ansible.builtin.copy` | Docs for a module |

Config precedence: `ANSIBLE_CONFIG` env → `./ansible.cfg` → `~/.ansible.cfg` → `/etc/ansible/ansible.cfg`.

## Common modules
| Module | Use |
|---|---|
| `apt` / `dnf` / `yum` / `package` | Manage packages |
| `service` / `systemd` | Manage services |
| `copy` / `template` / `file` | Manage files (template = Jinja2) |
| `user` / `group` | Manage accounts |
| `git` | Clone/checkout repos |
| `lineinfile` / `blockinfile` | Edit file contents |
| `command` / `shell` | Run commands (shell allows pipes) |
| `uri` | HTTP requests |
| `cron` | Manage cron jobs |
