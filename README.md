ansible-git
=========

An ansible role for managing the installation of `git` version control system.

More information on `git` shell can be found at:

  - [https://git-scm.com/](https://git-scm.com/)


Requirements
------------

This role installs the `git` package using the target system's package manager.

It also requires `ansible-core` 2.15 or newer, as the Ubuntu upstream `ppa` is
managed with `ansible.builtin.deb822_repository`.

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

Testing
-------

The role uses Molecule with the Docker driver. Install the test dependencies and
run the scenarios:

```bash
python3 -m pip install -r .config/requirements.txt
molecule test --all
```

The `default` scenario covers installing git from the Ubuntu upstream `ppa` and
managing per-user `~/.gitconfig` files. The `absent` scenario covers removing the
package and the upstream repository.

Lint the role from the repository root with:

```bash
ansible-lint
```

License
-------

Apache-2.0
