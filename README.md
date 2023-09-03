ansible-git
=========

An ansible role for managing the installation of `git` version control system.

More information on `git` shell can be found at:

  - [https://git-scm.com/](https://git-scm.com/)


Requirements
------------

This role requires `git`  package to be avalible in package manager on the target system.

Example Playbook
----------------

### 1. All Options

This example shows all possible options and their default values on ubuntu system where this role uses upstream git `ppa` for downloading latest version of git software.

```yaml
- hosts: all
  roles:
    - role: rehanone.git
      vars:
        git:
          debug: true
          upstream:
            repository: 'ppa:git-core/ppa'
            state: present
            manage: true
          package:
            name: git
            state: present
            manage: true
          users:
          - name: ray
            state: present
            email: 'test@example.com'
            fullname: 'Rehan Mahmood'
            gitconfig:
              color:
                ui: true
              push:
                default: simple
              pull:
                ff: only
              fetch:
                prune: true
              init:
                defaultBranch: master
```

### 2. Minimum Required Options

This example shows all minimum required options for using this role.

```yaml
- hosts: all
  roles:
    - role: rehanone.git
      vars:
        git:
          users:
          - name: ray
            email: 'test@example.com'
            fullname: 'Rehan Mahmood'
```

License
-------

Apache-2.0
