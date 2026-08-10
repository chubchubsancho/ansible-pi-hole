# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this is

A single-purpose **Ansible role** (`chubchubsancho.pi_hole`) that installs
[Pi-hole](https://pi-hole.net/) from source and renders its full
`/etc/pihole/pihole.toml` configuration from a Jinja2 template. It is published to
Ansible Galaxy on release. There is no application code — everything here is YAML,
Jinja2, and CI configuration.

Target platforms: Debian family (apt) and RedHat family (dnf/yum). CI exercises Debian
11/12/13, Ubuntu 22.04/24.04/26.04 (LTS only), AlmaLinux 9/10, and Fedora 42.

## Layout

```
defaults/main.yml          Only pi_hole_version (Renovate-managed). Lowest precedence.
vars/main.yml              pi_hole_repo, pi_hole_prerequisites, pi_hole_install_dir.
tasks/main.yml             Entry point: prerequisites -> install.
tasks/{debian,redhat}-prerequisites.yml
                           Selected dynamically by `ansible_os_family | lower`.
tasks/install.yml          Config drift detection, template render, git clone, installer.
handlers/main.yml          "Restart pihole-FTL" (defined; currently not notified by any task).
templates/etc/pihole/pihole.toml.j2
                           ~1800 lines. The heart of the role. Mirrors upstream
                           pihole.toml including all its comments.
meta/main.yml              Galaxy metadata (namespace chubchubsancho, role_name pi_hole).
molecule/                  Docker-based integration tests.
README.md                  User-facing reference for every pi_hole_* variable.
```

`.travis.yml` is legacy and no longer used; GitHub Actions is the live CI.

## How the role works

1. `tasks/main.yml` includes `{{ ansible_os_family | lower }}-prerequisites.yml`, so
   the file names are load-bearing — adding a platform means adding
   `tasks/<family>-prerequisites.yml`.
   - The Debian file uses `ansible.builtin.apt` (it needs `update_cache`); the RedHat
     file uses `ansible.builtin.package`, which resolves to dnf5 on Fedora 41+ and
     RHEL 10, dnf on RHEL 8/9, and yum on older releases. Don't pin it back to a
     concrete module — dnf5 systems have no `dnf-3` and the `yum` module fails there.
   - RedHat-family hosts also get `epel-release` (skipped on Fedora): Pi-hole's
     dependency package pulls in `dialog` and `lshw`, which the enterprise rebuilds
     ship only in EPEL.
2. `tasks/install.yml`:
   - `stat /opt/pihole` → `pihole_installed`. Every install step is guarded by
     `when: pihole_installed.stat.isdir is undefined`, i.e. **the clone and
     `basic-install.sh` only run on a host that has never had Pi-hole installed**.
     Config templating is *not* guarded and runs every time.
   - Config drift detection: renders the template to `/tmp/pihole.toml.new`, strips
     comments and trailing `### ...` annotations from both it and the live file with
     `grep`/`sed`, compares stdout, and only writes `/etc/pihole/pihole.toml` when the
     meaningful content differs. This exists because pihole-FTL rewrites the file at
     runtime (adding `### CHANGED` markers and stats), which would otherwise make the
     task report changed on every run.
   - Clones `pi_hole_repo` at `pi_hole_version` into `pi_hole_install_dir`
     (`/etc/.pihole`, depth 20), then runs `bash basic-install.sh --unattended` from
     `<install_dir>/automated install`.

## Variable conventions

- Every role variable is prefixed `pi_hole_`. Template variables follow the TOML path:
  `[dns] → pi_hole_dns_*`, `[webserver.api] → pi_hole_webserver_api_*`,
  `[files.log] → pi_hole_files_log_*`, and so on for `dhcp`, `resolver`, `database`,
  `misc`, `debug`.
- **Defaults live in the template, not in `defaults/main.yml`.** ~166 template
  variables are undefined by default and resolved with `| default(...)`. Only
  `pi_hole_version` is in `defaults/main.yml`.
- Two quoting idioms coexist in the template; match the surrounding lines rather than
  picking one:
  - value quoted by the template: `netmask = "{{ pi_hole_dhcp_netmask | default('') }}"`
  - quotes carried inside the default: `database = {{ pi_hole_files_database | default('"/etc/pihole/pihole-FTL.db"') }}`
- Booleans always end in `| lower` so Jinja's `True` becomes TOML `true`:
  `{{ pi_hole_dns_dnssec | default(false) | lower }}`.
- Lists use an `{% if ... is defined %}` / `{% else %}` pair that emits the loop or the
  literal upstream default (usually `[]`). Keep both branches:

  ```jinja
  {% if pi_hole_dns_rev_servers is defined %}
    revServers = [
      {% for rev_server in pi_hole_dns_rev_servers %}
      "{{ rev_server }}",
      {% endfor %}
    ]
  {% else %}
    revServers = []
  {% endif %}
  ```

- Upstream's explanatory comments are copied verbatim into the template. When syncing
  with a new pihole-FTL release, bring the comments across too — they are what users
  read in the rendered file.
- Every task carries `tags`, always including `pihole` plus a role-specific set
  (`installation`, `prerequisites`, `prepare`, `configuration`, `check`, `source`).
  Tag new tasks the same way.
- Tasks that modify the target system use `become: true` at the task level, in both
  `tasks/debian-prerequisites.yml` and `tasks/redhat-prerequisites.yml`. Read-only
  probes (the initial `stat` of `/opt/pihole`) do not.

## Keeping README, template, and defaults in sync

`README.md` is the variable reference and is expected to document every `pi_hole_*`
variable the template consumes. **Adding or renaming a template variable requires a
matching README entry in the same change.** Renovate also rewrites the version in
both `defaults/main.yml` and `README.md` via `customManagers` regexes, so do not
reformat these lines:

```yaml
# renovate: datasource=github-releases packageName=pi-hole/pi-hole
pi_hole_version: "v6.4.3"
```

```markdown
<!-- renovate: datasource=github-releases packageName=pi-hole/pi-hole -->
* `pi_hole_version` (default: "v6.4.3") — ...
```

(The README's `pi_hole_version` default has drifted behind `defaults/main.yml`; if you
touch that line, resync it.) `meta/main.yml` platform list has likewise drifted behind
the CI matrix — update it when adding platform support.

## Testing

Integration tests are Molecule + Docker. `image` and `version` are **required
environment variables** — `molecule.yml` interpolates them into the platform image:

```bash
python3 -m pip install molecule molecule-plugins[docker] ansible-lint yamllint
image=debian version=12 molecule test
image=debian version=12 molecule converge   # apply role without teardown
image=debian version=12 molecule destroy
```

Notes:
- The converge playbook is `molecule/resources/playbook.yml`, which imports the role by
  its **Galaxy name** `chubchubsancho.pi_hole` (not the directory name).
- The `ansible` verifier is disabled; `molecule/default/tests/test_default.py` is a
  testinfra file that is not wired into the run. Verification is effectively "the
  converge playbook succeeds".
- Containers run `/sbin/init` privileged with cgroup mounts because the Pi-hole
  installer needs systemd.
- `molecule/default/Dockerfile.j2` bootstraps each image by probing for a package
  manager, in order: `apt-get`, `dnf5`, `dnf`, `yum`. The `dnf5` branch must stay
  ahead of `dnf` — on Fedora 41+ and RHEL 10 `dnf` is a symlink to dnf5, and the
  dnf4 branch's `/usr/bin/dnf-3` does not exist there.
- Pi-hole itself supports rpm hosts natively: `basic-install.sh` builds and installs
  a `pihole-meta.rpm` via `rpmbuild`. v6 has no supported-OS gate, so the role does
  not need `PIHOLE_SKIP_OS_CHECK`.
- Molecule connects as the unprivileged `ansible` user (see
  `molecule/default/group_vars/all.yml`), so `become: true` is genuinely exercised
  rather than being a no-op against root. Keep it that way — switching to root would
  make every `become` in the role untested.
- RedHat-family images also need `hosts:` in `/etc/nsswitch.conf` rewritten to
  `files dns`. Their default puts `nss-resolve` ahead of `dns` with
  `[!UNAVAIL=return]`, so a systemd-resolved that answers but cannot resolve ends the
  lookup instead of falling through to Docker's resolver. That surfaces late and
  confusingly: the Pi-hole installer runs to completion, then `gravity.sh`'s
  `getent hosts` gate fails and aborts with "No DNS resolution available" after a
  120-second wait.
- On RedHat-family images the stock sudo PAM stack fails inside a container with
  "PAM account management error: Authentication service cannot retrieve
  authentication info", reported before the `NOPASSWD` rule is consulted. The
  Dockerfile replaces `/etc/pam.d/sudo` with a permit-only stack there, and then
  asserts `runuser -u ansible -- sudo -n true` at build time so a broken image fails
  during build rather than as an opaque converge error. Debian images are untouched.

Linting:

```bash
ansible-lint -q .
yamllint .
```

Both run as a dedicated `lint` job in `.github/workflows/molecule.yaml` and are
gating — keep them clean. They are *not* part of the `molecule test` sequence
(molecule's own `lint:` key in `molecule.yml` is vestigial and never executes), so a
green molecule run says nothing about lint.

`.yamllint` sets 2-space indentation, 160-char lines (warning), and forbids non-empty
`{ }` / `[ ]` flow style — use block style in YAML. `.ansible-lint` mocks the `pi_hole`
role and excludes `.github/`, `.cache/`, `.git/`.

## CI / release

- `.github/workflows/molecule.yaml` — two jobs on PRs and pushes to `master`: a `lint`
  job (`yamllint .`, then `ansible-lint -q .`) and the matrix `molecule test`. They run
  in parallel; both must pass.
- `.github/workflows/release.yaml` — release-please (manifest mode, `release-type:
  simple`) on `master`; maintains `CHANGELOG.md` and `.release-please-manifest.json`.
- `.github/workflows/publish.yaml` — on GitHub release, imports the role into Galaxy.

`CHANGELOG.md` and `.release-please-manifest.json` are **generated** — never hand-edit
them.

## Commits and branches

- Commit messages **must** follow [Conventional Commits](https://conventionalcommits.org/);
  release-please derives versions and changelog sections from them. Types in use:
  `feat`, `fix`, `perf`, `revert`, `docs`, `style`, `chore`, `refactor`, `test`, `build`,
  `ci`. Scopes seen in history: `template`, `config`, `pi-hole`, `readme`, `renovate`,
  `deps`, `molecule`.
  Examples: `fix(template): sync pihole.toml with pihole-FTL v6.7 schema`,
  `docs: add pi_hole_resolver_macnames to README`.
- A `feat:` on a Pi-hole major bump should carry `BREAKING CHANGE:` in the body
  (Renovate is configured to do this automatically).
- Default branch is `master`; PRs target it, one feature branch per change.

## Common tasks

**Bump the Pi-hole version** — edit `pi_hole_version` in `defaults/main.yml` and the
matching README line, keeping both Renovate marker comments intact. Renovate normally
does this; commit type is `fix(pi-hole):` for patches, `feat(pi-hole):` for minor/major.

**Sync the template with a new pihole-FTL schema** — diff upstream's generated
`pihole.toml` against `templates/etc/pihole/pihole.toml.j2`, add/remove keys and their
comment blocks, wire each new key to a `pi_hole_<section>_<key>` variable with the
upstream value as the `| default(...)`, and document it in README. Commit as
`fix(template): ...`.

**Add a config option** — three files change together: the template, `README.md`, and
(only if it needs a role-level default) `defaults/main.yml`.
