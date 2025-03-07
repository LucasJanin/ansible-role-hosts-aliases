Ansible Role: Hosts Aliases
=========================

This Ansible role creates SSH and VSCode Remote SSH aliases for hosts in your inventory, making it easier to connect to remote servers. It adds these aliases to a user's bash profile on a management host.

Requirements
------------

- Ansible 2.1 or higher
- A management host where the aliases will be created (typically your local machine)
- Remote hosts that you want to connect to via SSH and VSCode Remote SSH

Role Variables
--------------

Required variables:

```yaml
# Host information lists - these should be populated from inventory or vars
dev_hostnames: []         # List of hostnames to create aliases for
dev_ips: []               # List of IPs corresponding to hostnames
dev_alias: []             # List of short aliases for each host
dev_vspaths: []           # List of paths to open in VSCode for each host
dev_ansible_ssh_users: [] # List of SSH users for each host

# Management host information
user: "your_username"     # Username on the management host
host: "your_hostname"   # Hostname of the management host
home: "/path/to/home/"  # Home directory path on the management host
```

Optional variables with defaults:

```yaml
# File paths
bash: ".bashrc"                             # Bash profile file to modify
bash_hosts_aliases: ".bash_hosts_aliases" # File to store the aliases

# Editor configuration
editor_path: "code"                         # Path to the VSCode executable
create_editor_aliases: true                 # Whether to create VSCode aliases

# Comment for the aliases file
comment: "Hosts Aliases"
```

Dependencies
------------

None.

Example Playbook
----------------

```yaml
- name: Create host aliases on management host
  hosts: localhost
  gather_facts: false
  vars:
    management_user: "you"
    management_host: "your-mac"
    management_home: "/Users/you/"
  pre_tasks:
    - name: Set aliases from inventory
      set_fact:
        dev_alias: "{{ dev_alias + [hostvars[item].alias | default(item)] }}"
      loop: "{{ dev_hostnames }}"
      
    - name: Set vspaths from inventory
      set_fact:
        dev_vspaths: "{{ dev_vspaths + [hostvars[item].vspath | default('/home/' + hostvars[item].ansible_user + '/')] }}"
      loop: "{{ dev_hostnames }}"
      
    - name: Set SSH users from inventory
      set_fact:
        dev_ansible_ssh_users: "{{ dev_ansible_ssh_users + [hostvars[item].ansible_user | default('ansible')] }}"
      loop: "{{ dev_hostnames }}"
  tasks:
    - name: Include hosts aliases role
      include_role:
        name: ansible-role-hosts-aliases
      vars:
        user: "{{ management_user }}"
        host: "{{ management_host }}"
        home: "{{ management_home }}"
        bash: ".bashrc"
        bash_hosts_aliases: ".bash_hosts_aliases"
        create_editor_aliases: true
```

Inventory Example
----------------

Here's an example inventory file format that works with this role:

```ini
[all]
ansible     ansible_host="192.168.1.10"   alias="a"     vspath="/home/ansible/ansible"
docker      ansible_host="192.168.1.11"   alias="d"     vspath="/data/docker"
homepage    ansible_host="192.168.1.12"   alias="hp"    vspath="/opt/homepage/config"

[all:vars]
ansible_user="ansible"
```

After running the role, you'll have aliases like:

```bash
# SSH aliases
alias ssha='ssh ansible@ansible'
alias sshd='ssh ansible@docker'
alias sshhp='ssh ansible@homepage'

# VSCode aliases
alias vsa='code --folder-uri "vscode-remote://ssh-remote+ansible@ansible/home/ansible/ansible"'
alias vsd='code --folder-uri "vscode-remote://ssh-remote+ansible@docker/data/docker"'
alias vshp='code --folder-uri "vscode-remote://ssh-remote+ansible@homepage/opt/homepage/config"'
```

License
-------

MIT

Author Information
------------------

- Lucas Janin
- https://lucasjanin.com
- https://mastodon.social/@lucas3d
