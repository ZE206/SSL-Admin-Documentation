## Task 9

## Ansible setup

```bash
docker network create ansible-net
```
I then initialized the containers

    Control Node: The "brain" where Ansible is installed.
    Target1 & Target2: Managed nodes running openssh-server.


I need to generate rsa keypair on congtrol node which allows handshake without password prompt

```bash
ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa
ssh-copy-id root@target1
ssh-copy-id root@target2
```

### lab configuration

playbook for lab

```bash
---
- name: Azure Ubuntu 22.04 Lab Setup
  hosts: all
  become: yes
  # Critical for 1GB: Run tasks sequentially to prevent OOM
  serial: 1 
  gather_facts: yes 

  vars:
    required_packages: [python3, git, vim, htop, curl, wget, tmux, ufw]
    ssh_port: 22

  tasks:
    # 1. Memory-Safe Package Installation
    - name: Update apt and install packages
      apt:
        name: "{{ required_packages }}"
        state: present
        update_cache: yes
        install_recommends: no
        force_apt_get: yes 

    # 2. Global Shell & Editor Customization
    - name: Set global bash aliases
      copy:
        dest: /etc/profile.d/lab_vars.sh
        content: |
          alias ll='ls -alF'
          alias update='sudo apt update && sudo apt upgrade -y'
        mode: '0644'

    - name: Ensure Vim is the default editor
      alternatives:
        name: editor
        path: /usr/bin/vim.basic

    # 3. Azure-Specific Security (SSH Hardening)
    - name: Configure SSH Security
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "{{ item.regexp }}"
        line: "{{ item.line }}"
      loop:
        - { regexp: '^PermitRootLogin', line: 'PermitRootLogin no' }
        - { regexp: '^PasswordAuthentication', line: 'PasswordAuthentication no' }
      notify: Reload SSH

    # 4. UFW Implementation
    - name: Configure UFW (Firewall)
      ufw:
        state: enabled
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop:
        - "22"
        - "80"
        - "443"

  handlers:
    - name: Reload SSH
      service:
        name: ssh
        state: reloaded
```

run it

```bash
ansible-playbook -i hosts.ini lab_setup.yml --forks=1
```


Directory
```code
site.yml
inventory.ini
roles/
  ├── lab-base/
  │   ├── tasks/
  │   │   └── main.yml
  │   └── handlers/
  │       └── main.yml
  └── student-workstation/
      └── tasks/
          └── main.yml
```

first role is for making a base for the server to build on.
roles/lab-base/tasks/main.yml
```bash
---
- name: Install monitoring tools
  apt:
    name: [htop, curl, wget]
    state: present

- name: SSH Hardening - Allow Root via Key Only
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: "{{ item.regexp }}"
    line: "{{ item.line }}"
  loop:
    - { regexp: '^PermitRootLogin', line: 'PermitRootLogin prohibit-password' }
    - { regexp: '^PasswordAuthentication', line: 'PasswordAuthentication no' }
  notify: Restart SSH
```


second role is for creating a student account and deploying dev tools.
roles/student-workstation/tasks/main.yml
```bash
---
- name: Create a student user account
  user:
    name: student
    shell: /bin/bash
    groups: sudo
    append: yes

- name: Install Development Environments
  apt:
    name: [git, python3, vim, tmux]
    state: present

- name: Set Vim as default editor
  alternatives:
    name: editor
    path: /usr/bin/vim.basic
```

During the deployment, I hit a failure where PermitRootLogin no caused a lockout. To fix it, I updated the role to prohibit-password and ensured the SSH key was correctly placed in /root/.ssh/authorized_keys via the host-machine bypass.

To tie it all together, I created a master site.yml that applies both roles sequentially.

```bash
- name: Apply Lab Roles
  hosts: all
  become: yes
  serial: 1
  roles:
    - lab-base
    - student-workstation
```

to execute it all i did

```bash
ansible-playbook -i inventory.ini site.yml --forks=1
```