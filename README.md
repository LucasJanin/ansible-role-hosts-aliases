# Ansible Role: Hosts Aliases

This Ansible role creates SSH and VSCode Remote SSH aliases for hosts in your inventory, making it easier to connect to remote servers. It adds these aliases to a user's bash profile on a management host.

## Features

- Add all host to an bash_hosts_aliases
- Alias of ssh access to the host
- Alias of editor access to the host (optional)

## Screenshot
![Hosts Aliases](images/hosts-aliases.png )

## Requirements

- Ansible 2.1 or higher
- A management host where the aliases will be created (typically your local machine)
- Remote hosts that you want to connect to via SSH and VSCode Remote SSH

## Role Variables

Required management host information

| Variable | Type | Description | Default |
|----------|------|-------------|---------|
|user|str|Username on the management host||
|host|str|Hostname of the management host||
|home|str|Home directory path on the management host||

Optional variables with defaults:

| Variable | Type | Description | Default |
|----------|------|-------------|---------|
|bash|str|Bash profile file to modify|".bashrc"|
|bash_hosts_aliases|str|File to store the aliases|".bash_hosts_aliases"|
|editor_path|str|Path to the VSCode executable|"code"|
|create_editor_aliases|bool|Whether to create VSCode aliases|true|
|comment|str|Comment for the aliases file|"Hosts Aliases"|

## Dependencies

None

# New Method

This role now directly uses Ansible's host variables instead of requiring separate lists. This simplifies configuration and makes the role more maintainable.

The role now uses:
	•	ansible_play_hosts_all instead of dev_hostnames
	•	hostvars[item].ansible_host instead of dev_ips
	•	hostvars[item].alias instead of dev_alias
	•	hostvars[item].vspath instead of dev_vspaths
	•	hostvars[item].ansible_user instead of dev_ansible_ssh_users
This eliminates the need for pre-tasks that extract values from the inventory into separate lists, making your playbooks simpler and more maintainable.

## Example Playbook

```yaml
- name: Create host aliases on management host
  hosts: localhost
  gather_facts: false
  vars:
    management_user: "you"
    management_host: "your-mac"
    management_home: "/Users/you/"
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

## Inventory Example

Here's an example inventory file format that works with this role:

```ini
[all]
ansible     ansible_host="192.168.1.10"   alias="a"     vspath="/home/ansible/ansible"
docker      ansible_host="192.168.1.11"   alias="d"     vspath="/data/docker"
homepage    ansible_host="192.168.1.12"   alias="hp"    vspath="/opt/homepage/config"

[all:vars]
ansible_user="ansible"
```

## Testing

The role includes a test playbook and inventory that can be used to verify functionality. To run the tests:

```bash
cd ansible-role-hosts-aliases
ansible-playbook -i tests/inventory.ini tests/test.yml
```

The test creates bash aliases in a temporary directory to avoid modifying your actual bash profile during testing.

```bash
cat /tmp/test_bash_hosts_aliases
```

After running the role, you'll have aliases like:

```bash
# .bashrc_hosts_aliases
# Hosts Aliases

# ansible ---- 192.168.1.10
alias ssha='ssh ansible@ansible'
alias vsa='code --folder-uri "vscode-remote://ssh-remote+ansible@ansible/home/ansible/ansible"'

# docker ---- 192.168.1.11
alias sshd='ssh ansible@docker'
alias vsd='code --folder-uri "vscode-remote://ssh-remote+ansible@docker/data/docker"'

# homepage ---- 192.168.1.12
alias sshhp='ssh ansible@homepage'
alias vshp='code --folder-uri "vscode-remote://ssh-remote+ansible@homepage/opt/homepage/config"'
```

## License

MIT

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin feature/my-new-feature`)
5. Create a new Pull Request

## Author Information

Lucas Janin
- Mastodon: [https://mastodon.social/@lucas3d](https://mastodon.social/@lucas3d)
- Website: [https://www.lucasjanin.com](https://www.lucasjanin.com)
- GitHub: [github.com/lucasjanin](https://github.com/lucasjanin)
- LinkedIn: [linkedin.com/in/lucasjanin](https://linkedin.com/in/lucasjanin)