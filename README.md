# Ansible role for Users

 - manages users/groups
 - manages user's ssh keys
 - manages user's authorized keys

### Dependencies

* Ansible >= 2.4

### Installation

```console
cat <<'EOF' >> requirements.yaml
- src: git+git@gitlab.e-magnum.kz:devops/ansible/roles/users.git
  version: v0.1.0
EOF

ansible-galaxy install -r requirements.yaml -p roles
```

### Variables

Here is a list of all the default variables for this role, which are also available in `defaults/main.yml`.

```yaml
---
# This role takes advantage of Ansible's user role.
# All user related properties will fall back to Ansible's default values.
# @see http://docs.ansible.com/ansible/user_module.html
#
# users:
#   - username: foobar              (required)
#     name: Foo Bar
#     uid: 1000
#     group: staff
#     password: xxxxx               (a hash created with: mkpasswd)
#     groups: ["adm", "www-data"]
#     append: no                    (only append groups, leave others)
#     home_mode: "0750"
#     home_create: yes
#     home: /path/to/user/home
#     home_files:
#       - "/path/to/user/home/.bashrc"
#       - "/path/to/user/home/.bash_profile"
#     system: no
#     authorized_keys:
#       - "xxx"
#       - "{{ lookup('file', '/path/to/id_rsa.pub') }}"
#     authorized_keys_exclusive: yes
#     ssh_key_type: rsa
#     ssh_key_bits: 2048
#     ssh_key_password: ""
#     ssh_key_generate: no
#     ssh_key: "xxx" or "{{ lookup('file', '/path/to/id_rsa') }}"
#     ssh_keys:
#       id_rsa_1: "xxx" or "{{ lookup('file', '/path/to/id_rsa') }}"
#       id_rsa_2: "xxx" or "{{ lookup('file', '/path/to/id_rsa') }}"
#     shell: /bin/bash
#     update_password: always
#     user_create: yes
#
# users_remove:
#   - foobar
#   - { username: foobar, remove: no }

# list of users to add
users: []
# create the users
users_user_create: yes
# default user's dotfiles
users_home_files: []
# users home directory
users_home: /home
# create user's home directory
users_home_create: yes
# default user's primary group for users
users_group:
# default user's secondary groups
users_groups: []
# default user's home directory permissions
users_home_mode: "0755"
# default user login shell
#users_shell:
# default user's ssh key type
users_ssh_key_type: rsa
# default user's ssh key bits
users_ssh_key_bits: 2048
# default user's setting for authorized keys exclusive
users_authorized_keys_exclusive: no
# list of users to be removed
users_remove: []

```

### Usage

Example of simple user creation with password - playbook.yaml:

```yaml
---
- hosts: all
  roles:
    - users
  vars:
    users:
      - username: foobar
        name: Foo Bar
        password: LbCnhuZEV2vYI # test123 in hash format. Password created with mkpasswd command.
```

Example of user creation with authorized key - playbook.yaml:

```yaml
---
- hosts: all
  roles:
    - users
  vars:
    users:
      - username: foobar
        authorized_keys:
          - "{{ lookup('file', 'ssh_keys/id_rsa.pub') }}" # Place your public key to ssh_keys folder, near with your playbook
```

Example of user creation with appending to additional groups - playbook.yaml:

```yaml
---
- hosts: all
  roles:
    - users
  vars:
    users:
      - username: foobar
        groups:
          - test
          - docker
        append: yes
        authorized_keys:
          - "{{ lookup('file', 'ssh_keys/id_rsa.pub') }}" # Place your public key to ssh_keys folder, near with your playbook
```

Example of changing existing user params - playbook.yaml:

```yaml
---
- hosts: all
  roles:
    - users
  vars:
    users:
      - username: root
        home: /root
        group: root
        authorized_keys:
          - "{{ lookup('file', 'files/id_rsa.pub') }}"
        user_create: no # Important!!!
```

Example of user creation with generation ssh keys - playbook.yaml:

```yaml
---
- hosts: all
  roles:
    - users
  vars:
    users:
      - username: foobar
        ssh_key_generate: yes
        ssh_key_password: "" # set your passphrase, if you want. Empty by default.
```

Example of user removing - playbook.yaml:

```yaml
---
- hosts: all
  roles:
    - users
  vars:
    users_remove:
      - username: foobar
```


Example with different variations - playbook.yaml:

```yaml
---
- hosts: all
  roles:
    - users
  vars:
    users:
      - username: root
        home: /root
        group: root
        authorized_keys:
          - "{{ lookup('file', 'files/id_rsa.pub') }}"
        user_create: no
      - username: foobar
        name: Foo Bar 1
      - username: foobar_authorized_keys
        authorized_keys:
          - "{{ lookup('file', 'files/id_rsa.pub') }}"
        home_create: yes
      - username: foobar_nohome
        home_create: no
      - username: foobar_groups
        groups:
          - users
        append: yes
      - username: foobar_groups_reset
        groups: []
        group: foobar_groups_reset
      - username: foobar_home_mode
        home_mode: "0750"
      - username: foobar_key
        ssh_key: "{{ lookup('file', 'files/id_rsa') }}"
      - username: foobar_keys
        ssh_keys:
          id_rsa_1: "{{ lookup('file', 'files/id_rsa') }}"
          id_rsa_2: "{{ lookup('file', 'files/id_rsa') }}"
      - username: foobar_key_generate
        ssh_key_generate: yes
        ssh_key_password: secret
      - username: foobar_system
        system: yes
      - username: foobar_file
        home_files:
          - "files/.bashrc"
    users_group: staff
    users_groups:
      - www-data
    users_authorized_keys_exclusive: yes
    users_remove:
    - foobar
    - { username: foobar_key, remove: no }
    - { username: foobar_authorized_keys, remove: yes }
```

### Testing

Use molecule for testing this role:

```console
git clone https://gitlab.e-magnum.kz/devops/ansible/roles/users.git
cd users/
python3 -m venv venv
source venv/bin/activate
pip3 install --upgrade pip
pip3 install -r requirements.txt
```

Run molecule test:

```console
molecule test -s default
```

Run molecule test without destroy environment:

```console
molecule converge -s default
```
