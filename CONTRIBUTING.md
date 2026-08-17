# Contributing to ansible-role-local-repo

Thank you for your interest in contributing to this role!

This role points RHEL and Oracle Linux hosts at a local DNF/YUM repository
mirror for use in airgapped enclaves. It is deliberately small, and contributions
that keep it small and predictable are the most welcome kind.

## Code of Conduct

This project follows the [Contributor Covenant](CODE_OF_CONDUCT.md). Please be
respectful and constructive in all interactions.

## Development Environment Setup

### RHEL 9 Python Configuration

RHEL 9 system tools (like `subscription-manager`, `dnf`, etc.) require Python 3.9.
However, `ansible-core` 2.21+ requires Python 3.12+. To avoid breaking system tools,
we use a dual-Python setup:

| Command    | Version | Purpose                                    |
|------------|---------|------------------------------------------- |
| `python3`  | 3.9.x   | System tools (subscription-manager, dnf)  |
| `python`   | 3.12.x  | Development tools (ansible, pre-commit)   |

#### Installing Python 3.12 on RHEL 9

```bash
# Python 3.12 ships in the RHEL 9.4+ AppStream repository (no EPEL required)
sudo dnf install -y python3.12 python3.12-pip python3.12-devel

# Configure alternatives (keep python3 pointing to 3.9 for system tools)
sudo alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 3
sudo alternatives --set python3 /usr/bin/python3.9

# Set python (without the 3) to use 3.12 for development
sudo alternatives --install /usr/bin/python python /usr/bin/python3.12 2
sudo alternatives --set python /usr/bin/python3.12

# Verify configuration
python3 --version   # Should show Python 3.9.x
python --version    # Should show Python 3.12.x
```

> **WARNING**: Do NOT change `python3` to point to Python 3.12. This will break
> `subscription-manager`, `dnf`, and other RHEL system tools. Recovery requires
> manually creating a temporary repo with entitlement certificates to reinstall
> `subscription-manager`. Don't ask how we know this.

#### Installing Development Dependencies

All Python dependencies are hash-pinned in `requirements.txt`, which is generated
from `requirements.in` via `pip-compile --generate-hashes` (pip-tools) for
reproducible, integrity-verified installs across local development and CI/CD. Edit
`requirements.in` (not `requirements.txt`) and recompile to change a dependency;
Dependabot maintains the lock automatically.
Choose **one** of the following installation methods:

##### Option A: System-wide Installation (Simpler)

Install tools directly using the `python` (3.12) command:

```bash
# Upgrade pip first
python -m pip install --upgrade pip

# Install all dev dependencies from requirements.txt
python -m pip install --require-hashes -r requirements.txt
```

##### Option B: Virtual Environment (Isolated)

Use a venv to isolate development dependencies from the system:

```bash
# Create a virtual environment with Python 3.12
python3.12 -m venv ~/.venv/ansible-dev

# Add activation alias to your shell (optional convenience)
echo 'alias ansible-dev="source ~/.venv/ansible-dev/bin/activate"' >> ~/.bashrc
source ~/.bashrc

# Activate the virtual environment
source ~/.venv/ansible-dev/bin/activate
# Or use the alias: ansible-dev

# Install dependencies inside the venv
pip install --upgrade pip
pip install --require-hashes -r requirements.txt

# Deactivate when done
deactivate
```

> **Note**: When using a venv, you must activate it before running any development
> commands (ansible-lint, pre-commit, etc.).

##### Verify Installation

Regardless of which method you chose:

```bash
ansible --version      # Should show Python 3.12.x
ansible-lint --version
yamllint --version
pre-commit --version
```

### Pre-commit Hooks

This repository uses pre-commit hooks to ensure code quality before commits:

```bash
# Install hooks (one-time setup)
pre-commit install

# Run manually on all files
pre-commit run --all-files

# Hooks will run automatically on git commit
```

The following hooks are configured:
- **yamllint** - YAML syntax and style validation
- **ansible-lint** - Ansible best practices (production profile)
- **trailing-whitespace** - Remove trailing whitespace
- **end-of-file-fixer** - Ensure files end with newline
- **check-yaml** - Validate YAML syntax
- **check-added-large-files** - Prevent large files (>500KB)
- **check-merge-conflict** - Detect merge conflict markers
- **detect-private-key** - Prevent accidental key commits

### Testing

This role has no Molecule suite. The full local test cycle is the same three
checks CI runs:

```bash
yamllint .
ansible-lint
ansible-playbook tests/test.yml -i tests/inventory --syntax-check
```

`ansible-lint` runs the `production` profile with `strict: true`, so warnings
fail the run. All three must pass before a pull request will be merged.

> **Note on syntax-check coverage**: `tasks/main.yml` dispatches to the
> platform files via `include_tasks`, which is dynamic. `--syntax-check`
> therefore validates the play and `tasks/main.yml` but does not descend into
> `redhat.yml` or `oracle.yml`. Those files are fully parsed by `ansible-lint`,
> so the role has no blind spot, but do not treat a passing syntax check alone
> as proof a platform task file is valid.

## How to Contribute

### Reporting Issues

- Check existing issues before creating a new one
- Include the target platform (RHEL 8/9/10 or Oracle Linux 9), the
  `ansible-core` version, and the relevant role variables
- Redact internal hostnames and enclave identifiers before posting
- Provide steps to reproduce the issue

Security vulnerabilities must **not** be reported as public issues. See
[SECURITY.md](SECURITY.md) for the private reporting process.

### Submitting Changes

1. **Fork** the repository
2. **Create a branch** for your changes:
   ```bash
   git checkout -b fix/short-description
   ```
3. **Follow the code style** documented below
4. **Test your changes**:
   ```bash
   yamllint .
   ansible-lint
   ansible-playbook tests/test.yml -i tests/inventory --syntax-check
   ```
5. **Commit** using [Conventional Commits](https://www.conventionalcommits.org/):
   ```bash
   git commit -m "fix: correct the Oracle Linux AppStream base URL"
   ```
   Prefixes in use: `feat`, `fix`, `docs`, `ci`, `deps`, `chore`, `refactor`.
   Dependabot uses `ci` for GitHub Actions bumps and `deps` for pip bumps.
6. **Push** and open a Pull Request

### Code Style

- **Classification banner.** Every YAML and configuration file carries the
  standard header block immediately after the `---` document marker:
  ```yaml
  ---
  ###############################################################################
  # Filename: tasks/example.yml
  # Role: ansible-role-local-repo
  # Summary: One-line description of what this file does
  # Last Updated: DD Mon YYYY
  # Classification: UNCLASSIFIED
  ###############################################################################
  ```
  Update `Last Updated` when you change a file. This is not decoration. The
  repository is reviewed against DoD marking expectations, and unmarked files
  are treated as a defect.
- **Use FQCN for all modules** (`ansible.builtin.yum_repository`, not
  `yum_repository`). This is enforced by `ansible-lint`.
- **Comments wrap at 80 characters.**
- **Explain the *why*, not the *what*.** The existing comments document
  non-obvious constraints (why `nofilter` is set on actionlint, why
  `--syntax-check` does not descend into the platform files). Match that bar.
- **No secrets, internal hostnames, enclave names, or unit identifiers** in
  code, comments, commit messages, or test fixtures. Defaults must use
  non-routable placeholder values.

### Adding a New Platform

1. Add a task file under `tasks/` named for the distribution (e.g. `suse.yml`)
2. Add a dispatch entry in `tasks/main.yml` keyed on
   `ansible_facts['distribution']`
3. Add any new variables to `defaults/main.yml` with a sensible,
   non-routable default
4. Add the platform to `galaxy_info.platforms` in `meta/main.yml`
5. Document the new variables in the README's Role Variables section
6. Keep `gpgcheck: true` and an explicit `gpgkey` on every repository you define

### Changing Defaults

Repository URLs, GPG key paths, and the transport protocol are security-relevant.
When changing a default:

- Never weaken `gpgcheck` or remove a `gpgkey`
- Never default `web_protocol` to `http`
- Never use a publicly registrable hostname as a default; use a reserved
  suffix such as `.local` or `.invalid`

## Questions?

Open an issue with the `question` label.
