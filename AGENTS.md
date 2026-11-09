# AGENTS.md

## What this is

Ansible role `rehanone.git` (namespace `rehanone`, role `git` in `meta/main.yml`). It installs git and renders per-user `~/.gitconfig`. No Python/JS build; work happens in YAML/Jinja. Supported distros are listed in `meta/main.yml`.

## Commands

- Lint (the CI gate): `ansible-lint` from the repo root. Config and ignore file are auto-discovered from `.config/ansible-lint.yml` and `.config/ansible-lint-ignore.txt`.
- Syntax check: `ansible-playbook -i tests/inventory tests/test.yml --syntax-check`
- `tests/test.yml` references role `rehanone.git`. To exercise the working tree, run with `ANSIBLE_ROLES_PATH=.ansible/roles`; `.ansible/roles/rehanone.git` is a symlink to this repo. The copy in `~/.ansible/roles/rehanone.git` is an installed snapshot and may be stale.
- There is no Makefile/tox/pytest. A real test run targets `localhost`/`root` and mutates the host (adds apt PPA, installs git, writes `~/.gitconfig`), so prefer `--syntax-check`/`--list-tasks` unless changes are intended.

## Layout and data flow

- `tasks/main.yml` only includes `tasks/git.yml`; all logic lives in `tasks/git.yml`.
- Variable layering: `defaults/main.yml` scalar `git_*` defaults build `git_defaults`/`git_user_defaults` in `vars/main.yml`; the playbook's nested `git:` dict (see README) is recursively combined over those to produce `git_config`.
- Distribution vars load first via `first_found`: `vars/distribution/<ansible_facts.distribution>.yml`, falling back to `vars/distribution/default.yml`. New distro support requires a file matching the exact fact value (e.g. `Ubuntu.yml`).
- Only `vars/distribution/Ubuntu.yml` enables the upstream `ppa:git-core/ppa`; all other distros use their package manager. The PPA is written with `deb822_repository` (not the deprecated `apt_repository`), using the Launchpad API to resolve the signing-key fingerprint and installing `python3-debian`/`ca-certificates` on Debian-family targets.
- User entries in `git.users` are merged with `git_user_defaults` (`manage`, `state`, `gitconfig`). `state: present` + `manage: true` writes `~<user>/.gitconfig` from `templates/gitconfig.j2`; `state: absent` removes it.
- Set `git_debug: true` (or `git.debug: true`) to print the fully merged `git_config`.

## Conventions and gotchas

- Keep the `# code: language=ansible` + `---` header on YAML files.
- Use FQCN modules (`ansible.builtin.*`); ansible-lint runs the `production` profile.
- `templates/gitconfig.j2` lowercases every value and always emits the `[user]` section from `fullname`/`email`.
- Use `become: true` for package/template/file tasks; the role assumes privilege escalation is available.
- `.gitattributes` enforces LF for `*.j2` and YAML files; do not introduce CRLF.
- Update `CHANGELOG.md` for user-visible changes (`## x.y.z (date)` + `**Features:**`).
- CI: `.github/workflows/ansible-lint.yml` runs ansible-lint on every push/PR branch; Dependabot tracks pip deps in `/.config/` and GitHub Actions.
