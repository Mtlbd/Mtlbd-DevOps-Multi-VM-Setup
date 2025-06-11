# DevOps Multi-VM Setup

This repository contains Ansible configuration and roles for provisioning a DevOps environment across three Ubuntu VMs: one as the Ansible controller, one for CI (Jenkins), and one for monitoring (Prometheus/Grafana).

## 📁 Directory Structure

```plaintext
devops-multi-vm/
├── ansible.cfg            # Ansible configuration (roles_path, inventory)
├── inventories/
│   └── hosts.ini          # Inventory file defining VMs and groups
├── playbooks/
│   └── site.yml           # Main playbook to run roles
└── roles/
    ├── docker/            # Role to install & configure Docker
    │   └── tasks/
    │       └── main.yml
    ├── jenkins/           # Role to deploy Jenkins container
    │   └── tasks/
    │       └── main.yml
    └── monitoring/        # Role to deploy Prometheus, Grafana, Node Exporter
        └── tasks/
            └── main.yml
```

## ⚙️ Configuration Files

### ansible.cfg

```ini
[defaults]
roles_path = ./roles
inventory   = ./inventories/hosts.ini
```

### inventories/hosts.ini

```ini
[all]
vm-ci    ansible_host=192.168.181.129 ansible_user=mt ansible_ssh_private_key_file=~/.ssh/id_ansible
vm-mon   ansible_host=192.168.181.130 ansible_user=mt ansible_ssh_private_key_file=~/.ssh/id_ansible

[ci]
ci-jenkins

[monitoring]
prom-monitoring
```

### playbooks/site.yml

```yaml
- name: Install Docker on all nodes
  hosts: all
  become: true
  roles:
    - docker

- name: Provision CI server (Jenkins)
  hosts: ci
  become: true
  roles:
    - jenkins

- name: Provision Monitoring server
  hosts: monitoring
  become: true
  roles:
    - monitoring
```

## 🚀 Prerequisites

1. **Ubuntu 22.04** on each VM
2. **SSH access** from controller (mt\@controller) to each node without password, using a key:

   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/id_ansible -N ""
   ssh-copy-id -i ~/.ssh/id_ansible.pub mt@192.168.181.129
   ssh-copy-id -i ~/.ssh/id_ansible.pub mt@192.168.181.130
   ```
3. **Passwordless sudo** for user `mt` on vm-ci and vm-mon:

   ```bash
   echo "mt ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/mt
   ```
4. **Ansible** installed on controller:

   ```bash
   sudo apt update
   sudo apt install -y software-properties-common
   sudo add-apt-repository --yes --update ppa:ansible/ansible
   sudo apt install -y ansible
   ```

## ▶️ Running the Playbook

From the root of the repository (`devops-multi-vm`):

```bash
ansible-playbook playbooks/site.yml
```

If you prefer to enter a sudo password interactively instead of passwordless sudo, add `-K`:

```bash
ansible-playbook playbooks/site.yml -K
```

## 🖥 Accessing Services

* **Jenkins**: [http://192.168.181.129:8080]
* **Prometheus**: [http://192.168.181.130:9090]
* **Grafana**: [http://192.168.181.130:3000] (default credentials: `admin`/`admin`)

---

For any issues or questions, please open an issue or contact the maintainer. Good luck with your DevOps setup!

---

> Created by [Mohammad Talebi](https://linkedin.com/in/mtlbd) – DevOps Engineer 👨‍💻

