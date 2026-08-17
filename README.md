# Ansible Role: Assign Local Repository

Sets the DNF repository for an airgap'd Enclave with no access to
Internet repositories.

## Requirements

None

## Role Variables

You can modify any of the following variables as you wish in the role's `defaults/main.yml`:

* `local_repo_server`: FQDN of the Yum Repository Server (Default: `repo.closednetwork.local`)
* `local_baseos_repo_folder`: Path to the RHEL Base OS Repo(Default: `rhel-10-for-x86_64-baseos-rpms`)
* `local_appstream_repo_folder`: Path to the RHEL AppStream Repo(Default: `rhel-10-for-x86_64-appstream-rpms`)
* `oracle9_baseos_repo_folder`: Path to the Oracle Linux 9 Base OS Repo(Default: `ol9_baseos_latest`)
* `oracle9_appstream_repo_folder`: Path to the Oracle Linux 9 AppStream Repo(Default: `ol9_appstream`)
* `oracle9_gpg_key`: GPG Path to the Oracle Linux 9 Repo (Default: `file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle`)
* `web_protocol`: Which web protocol to use for repository server (Default: `https`)

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
      local_baseos_repo_folder: rhel-9-for-x86_64-baseos-rpms
      local_appstream_repo_folder: rhel-9-for-x86_64-appstream-rpms
      web_protocol: https
```

## License

Apache License 2.0. See [LICENSE](LICENSE) for the full text.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development setup, testing, and code
style. All participants are expected to follow the
[Code of Conduct](CODE_OF_CONDUCT.md).

To report a security vulnerability, follow the private disclosure process in
[SECURITY.md](SECURITY.md) — please do not open a public issue.

## Author Information

Alex Ackerman, GitHub @darkhonor
