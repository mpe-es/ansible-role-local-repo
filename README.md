# Ansible Role: Assign Local Repository

Sets the DNF repository for an airgap'd Enclave with no access to
Internet repositories.

## Requirements

None

## Role Variables

You can modify any of the following variables as you wish in the role's `defaults/main.yml`:

* `local_repo_server`: FQDN of the Yum Repository Server (Default: `repo.closednetwork.local`)
* `local_repo_baseos_folder`: Path to the RHEL Base OS Repo(Default: `rhel-10-for-x86_64-baseos-rpms`)
* `local_repo_appstream_folder`: Path to the RHEL AppStream Repo(Default: `rhel-10-for-x86_64-appstream-rpms`)
* `local_repo_oracle9_baseos_folder`: Path to the Oracle Linux 9 Base OS Repo(Default: `ol9_baseos_latest`)
* `local_repo_oracle9_appstream_folder`: Path to the Oracle Linux 9 AppStream Repo(Default: `ol9_appstream`)
* `local_repo_oracle9_gpg_key`: GPG Path to the Oracle Linux 9 Repo (Default: `file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle`)
* `local_repo_web_protocol`: Which web protocol to use for repository server (Default: `https`)
* `local_repo_configure_cert_authority`: Install the enclave trust anchors
  before the repository files are written (Default: `false`)
* `local_repo_enclave_ca_certificates`: List of trust anchors to install. Each
  entry takes `name` (written to
  `/etc/pki/ca-trust/source/anchors/<name>.pem`), `url`, and an optional
  `checksum` in `get_url` form, e.g. `sha256:<digest>` (Default: a single
  `enclave-ca` entry served from `local_repo_server`)

### Upgrading from 1.x

Version 2.0.0 renamed every unprefixed role variable to the `local_repo_`
prefix. Rename these in your inventory, playbooks, and `include_role` `vars:`:

| 1.x name | 2.0.0 name |
|---|---|
| `local_baseos_repo_folder` | `local_repo_baseos_folder` |
| `local_appstream_repo_folder` | `local_repo_appstream_folder` |
| `oracle9_baseos_repo_folder` | `local_repo_oracle9_baseos_folder` |
| `oracle9_appstream_repo_folder` | `local_repo_oracle9_appstream_folder` |
| `oracle9_gpg_key` | `local_repo_oracle9_gpg_key` |
| `web_protocol` | `local_repo_web_protocol` |
| `configure_local_cert_authority` | `local_repo_configure_cert_authority` |
| `enclave_ca_certificates` | `local_repo_enclave_ca_certificates` |

Every entry point of the role, including `tasks_from`, first runs a guard
(`tasks/assert-renamed.yml`) that fails if any of the first six old names is
defined anywhere in scope: inventory, extra vars, `set_fact`, or another
role's defaults. Rename or remove such a variable even if it belongs to a
different role.

The two CA names are not rejected by name, because a calling role may own
them. Instead, `tasks/configure-ca.yml` refuses to install anchors while
`enclave_ca_certificates` is set and differs from
`local_repo_enclave_ca_certificates`; the role would otherwise install
`local_repo_enclave_ca_certificates` (by default an unpinned anchor) in place
of yours. A 1.x caller that still sets
`configure_local_cert_authority: true` through the main entry point, without
the new flag, silently gets CA trust off; the first HTTPS repository access
then fails with a TLS error against the mirror.

## Enclave Certificate Authority Trust

When the repository mirror serves HTTPS with a certificate issued by an enclave
CA, set `local_repo_configure_cert_authority: true`. The role then installs
each anchor in `local_repo_enclave_ca_certificates` and runs
`update-ca-trust extract` before writing any repository file, so DNF has a
trust path the first time it runs.

Two operational notes:

* The anchors are fetched with `validate_certs: false`. There is nothing to
  validate against on a first run, which is the trust bootstrap problem itself.
  **Set `checksum` on every entry.** The pin is what replaces TLS validation;
  `get_url` fails closed when the digest does not match. Without it, whoever
  controls that first response controls every TLS decision the host makes
  afterward. Where the anchor can be staged on the Ansible control node
  instead, that channel is already authenticated and is the stronger option.
* `update-ca-trust extract` is the only supported form on RHEL 10. The legacy
  `enable` argument was removed there and exits non-zero; on Oracle Linux 9 it
  still runs but prints a deprecation warning.

## Dependencies

None

## Example Playbook

Here is an example playbook using this role:

```yaml
- name: Apply Baseline Configuration
  become: true
  become_method: sudo
  gather_facts: true
  hosts: all
  roles:
    - role: local_repo
      local_repo_server: mirror.securenet.local:9795
      local_repo_baseos_folder: rhel-9-for-x86_64-baseos-rpms
      local_repo_appstream_folder: rhel-9-for-x86_64-appstream-rpms
      local_repo_web_protocol: https
      local_repo_configure_cert_authority: true
      local_repo_enclave_ca_certificates:
        - name: enclave-root-ca
          url: https://mirror.securenet.local:9795/certs/root-ca.pem
          checksum: sha256:6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b
        - name: enclave-issuing-ca
          url: https://mirror.securenet.local:9795/certs/issuing-ca.pem
          checksum: sha256:d4735e3a265e16eee03f59718b9b5d03019c07d8b6c51f90da3a666eec13ab35
```

## License

Apache License 2.0. See [LICENSE](LICENSE) for the full text.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development setup, testing, and code
style. All participants are expected to follow the
[Code of Conduct](CODE_OF_CONDUCT.md).

To report a security vulnerability, follow the private disclosure process in
[SECURITY.md](SECURITY.md). Please do not open a public issue.

## Author Information

Alex Ackerman, GitHub @darkhonor
