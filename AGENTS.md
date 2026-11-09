# AGENTS.md

## What this is

Ansible role `rehanone.git` (namespace `rehanone`, role `git` in `meta/main.yml`). It installs git and renders per-user `~/.gitconfig`. No Python/JS build; work happens in YAML/Jinja. Supported distros are listed in `meta/main.yml`.

## Commands

- Lint (the CI gate): `ansible-lint` from the repo root. Config and ignore file are auto-discovered from `.config/ansible-lint.yml` and `.config/ansible-lint-ignore.txt`.
- Tests: install `python3 -m pip install -r .config/requirements.txt`, then run `molecule test --all` (Docker required). Scenarios live in `molecule/default` (present) and `molecule/absent` (removal).
- Syntax check: `ansible-playbook -i tests/inventory tests/test.yml --syntax-check`
- `tests/test.yml` references role `rehanone.git`. To exercise the working tree, run with `ANSIBLE_ROLES_PATH=.ansible/roles`; `.ansible/roles/rehanone.git` is a symlink to this repo. The copy in `~/.ansible/roles/rehanone.git` is an installed snapshot and may be stale.
- `tests/test.yml` is legacy. Molecule is the real test; it targets a disposable `ubuntu:jammy` container and does not mutate the host.

## Layout and data flow

- `tasks/main.yml` only includes `tasks/git.yml`; all logic lives in `tasks/git.yml`.
- Variable layering: `defaults/main.yml` scalar `git_*` defaults build `git_defaults`/`git_user_defaults` in `vars/main.yml`; the playbook's nested `git:` dict (see README) is recursively combined over those to produce `git_config`.
- Distribution vars load first via `first_found`: `vars/distribution/<ansible_facts.distribution>.yml`, then `vars/distribution/<ansible_facts.os_family>.yml`, falling back to `vars/distribution/default.yml`. New distro support requires a file matching the exact fact value (e.g. `Ubuntu.yml`).
- Only `vars/distribution/Ubuntu.yml` enables the upstream `ppa:git-core/ppa`; all other distros use their package manager. The PPA is written with `deb822_repository` (not the deprecated `apt_repository`): the Launchpad API resolves the signing-key fingerprint, `get_url` downloads the key to `/etc/apt/keyrings/`, and `python3-debian`/`ca-certificates` are installed on Debian-family targets.
- User entries in `git.users` are merged with `git_user_defaults` (`manage`, `state`, `gitconfig`). `state: present` + `manage: true` writes `~<user>/.gitconfig` from `templates/gitconfig.j2`; `state: absent` removes it. Names must be unique; present managed users must define `fullname` and `email`.
- Set `git_debug: true` (or `git.debug: true`) to print the fully merged `git_config`.
- `meta/runtime.yml` and `meta/argument_specs.yml` describe the minimum Ansible version and the top-level role options. The argument spec deliberately does not describe the nested `git` dict.

## Conventions and gotchas

- Keep the `# code: language=ansible` + `---` header on YAML files.
- Use FQCN modules (`ansible.builtin.*`); ansible-lint runs the `production` profile.
- `templates/gitconfig.j2` preserves value casing and only normalises booleans; it always emits the `[user]` section from `fullname`/`email`.
- Use `become: true` for package/template/file tasks; the role assumes privilege escalation is available.
- `.gitattributes` enforces LF for `*.j2` and YAML files; do not introduce CRLF.
- Update `CHANGELOG.md` for user-visible changes (`## x.y.z (date)` + `**Features:**`).
- CI: `.github/workflows/ansible-lint.yml` runs ansible-lint and `.github/workflows/molecule.yml` runs both scenarios on every push/PR branch; Dependabot tracks pip deps in `/.config/` and GitHub Actions.
