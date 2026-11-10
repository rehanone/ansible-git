## 0.0.4 (November 09, 2026)

**Features:**

- Update the supported platform matrix for current EL, Fedora, openSUSE,
  Ubuntu, Debian, and FreeBSD releases.

## 0.0.3 (November 08, 2026)

**Features:**

- Add Molecule scenarios for the present and absent states, and a GitHub
  Actions workflow to run them.

**Fixes:**

- Preserve case in git configuration values; only booleans are normalised.
- Validate user names, duplicate users, and required `fullname`/`email` values.
- Make the upstream signing-key download idempotent.
- Add runtime metadata and role argument specs.

## 0.0.2 (November 08, 2026)

**Fixes:**

- Replace the deprecated `ansible.builtin.apt_repository` module with
  `ansible.builtin.deb822_repository` for the Ubuntu upstream `ppa`.

## 0.0.1 (September 18, 2025)

**Features:**

- Initial release.
