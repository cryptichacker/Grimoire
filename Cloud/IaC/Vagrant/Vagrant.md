---
tags: [cloud, iac, vagrant]
type: cheatsheet
source: Vagrant CLI reference (compiled)
last-verified: 2026-08-25
---

# Vagrant

## Up
- [[IaC]]

Vagrant builds and manages reproducible development VMs (and containers) from a single `Vagrantfile`, on top of **providers** like VirtualBox, libvirt, VMware, Hyper-V, or Docker.

## Lifecycle
| Command | Description |
|---|---|
| `vagrant init <box>` | Create a `Vagrantfile` (e.g. `vagrant init ubuntu/jammy64`) |
| `vagrant up` | Create/boot the machine(s) and provision on first boot |
| `vagrant up --provider=libvirt` | Boot with a specific provider |
| `vagrant halt` | Graceful shutdown |
| `vagrant suspend` / `vagrant resume` | Save/restore VM state to disk |
| `vagrant reload` | Restart, re-reading the `Vagrantfile` |
| `vagrant reload --provision` | Restart and re-run provisioners |
| `vagrant destroy` | Delete the machine(s) (`-f` to skip confirm) |
| `vagrant status` | Status of machines in this environment |
| `vagrant global-status` | All Vagrant machines on the host (`--prune` to clean stale) |

## Access & provisioning
| Command | Description |
|---|---|
| `vagrant ssh` | SSH into the (default) machine |
| `vagrant ssh <name>` | SSH into a named machine (multi-machine) |
| `vagrant ssh-config` | Print SSH connection settings |
| `vagrant provision` | Re-run provisioners on a running machine |
| `vagrant up --no-provision` | Boot without provisioning |
| `vagrant port` | Show forwarded ports |
| `vagrant validate` | Validate the `Vagrantfile` |

## Boxes
| Command | Description |
|---|---|
| `vagrant box add <name>` | Download a box (from Vagrant Cloud) |
| `vagrant box list` | List installed boxes |
| `vagrant box outdated` | Check for updates |
| `vagrant box update` | Update the current box |
| `vagrant box remove <name>` | Delete a box |
| `vagrant package` | Package the running VM into a reusable `.box` |

## Snapshots
| Command | Description |
|---|---|
| `vagrant snapshot save <name>` | Take a named snapshot |
| `vagrant snapshot restore <name>` | Roll back to a snapshot |
| `vagrant snapshot list` | List snapshots |
| `vagrant snapshot delete <name>` | Delete a snapshot |
| `vagrant snapshot push` / `pop` | Stack-style snapshot save/restore |

## Plugins
| Command | Description |
|---|---|
| `vagrant plugin install <name>` | Install a plugin (e.g. `vagrant-libvirt`, `vagrant-disksize`) |
| `vagrant plugin list` | List installed plugins |
| `vagrant plugin update` / `uninstall <name>` | Update / remove |

## Vagrantfile essentials
```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.hostname = "dev"

  # Networking
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Shared folder (host ./ -> guest /vagrant by default)
  config.vm.synced_folder "./app", "/srv/app"

  # Provider tuning
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 2048
    vb.cpus   = 2
  end

  # Provisioning
  config.vm.provision "shell", inline: "apt-get update && apt-get install -y nginx"
  config.vm.provision "ansible" do |a|
    a.playbook = "provision.yml"
  end
end
```

## Multi-machine
```ruby
Vagrant.configure("2") do |config|
  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/jammy64"
  end
  config.vm.define "db" do |db|
    db.vm.box = "ubuntu/jammy64"
  end
end
```
Then target individual machines: `vagrant up web`, `vagrant ssh db`, `vagrant destroy -f`.

## Handy flags
| Flag | Description |
|---|---|
| `--provider=<name>` | Choose provider (virtualbox, libvirt, docker, hyperv, vmware_desktop) |
| `--provision` / `--no-provision` | Force / skip provisioning |
| `--provision-with <name>` | Run only the named provisioner |
| `-f` / `--force` | Skip confirmation (e.g. destroy) |
| `VAGRANT_LOG=info` | Verbose logging (env var) |
