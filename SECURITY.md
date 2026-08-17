# Security Policy

## Reporting a Vulnerability

The `ansible-role-local-repo` project takes security seriously. Because this
role rewrites DNF/YUM repository configuration — including the base URLs and GPG
key paths that govern where packages come from and how they are verified — on
hosts intended for airgapped DoD enclaves, vulnerability handling is treated as
a high priority.

**Do not report security vulnerabilities through public GitHub issues, pull
requests, or discussions.**

### How to Report

Report vulnerabilities privately through GitHub's built-in **private
vulnerability reporting**:

1. Go to the repository's **Security** tab.
2. Select **Report a vulnerability** (under *Advisories*).
3. Complete the advisory form with the details below.

This opens a private security advisory visible only to the reporter and the
project maintainer ([@darkhonor](https://github.com/darkhonor)); it is never
publicly visible unless and until a fix is published. If you require encrypted
communication beyond the private advisory channel, request a public key in your
initial report and one will be provided out-of-band.

### What to Include

Please include as much of the following information as possible to help us
understand and reproduce the issue:

- The role version (release tag or git commit hash)
- The target platform (RHEL 8/9/10 or Oracle Linux 9)
- The Ansible / ansible-core version used to run the role
- The repository server configuration in use (`local_repo_server`,
  `web_protocol`, and the repo folder variables), with any internal hostnames
  redacted
- A description of the vulnerability and its potential impact
- Steps to reproduce the issue
- Any proof-of-concept code or playbook (please mark clearly as such)
- Suggested mitigations or fixes if you have them

Because this role controls package provenance, reports involving GPG
verification bypass, repository URL injection, or downgrade of transport
security are of particular interest.

### What to Expect

You can expect the following response from the maintainer:

| Phase | Target Time |
| ----- | ----------- |
| Initial acknowledgment | Within 3 business days |
| Initial assessment and severity rating | Within 7 business days |
| Patch development for confirmed Critical/High issues | Within 30 days |
| Patch development for confirmed Medium issues | Within 90 days |
| Coordinated disclosure (if applicable) | After patch release |

For DoD enclave deployments, vulnerability information may need to be handled
under controlled channels per the relevant Information System Security Officer
(ISSO) and Authorizing Official (AO) guidance. The maintainer will coordinate
with the reporter on appropriate handling.

## Supported Versions

`ansible-role-local-repo` follows [Semantic Versioning](https://semver.org/).
Security fixes are applied to the most recent release.

| Version | Supported |
| ------- | --------- |
| 0.x.x | Active development |

Once the project reaches a stable 1.0.0 release, this matrix will be updated to
reflect the supported version policy.

## Security Practices

This project follows these supply chain and code security practices:

- **Dependency monitoring:** Dependabot maintains the Python toolchain (pip) and
  the GitHub Actions used in CI.
- **Supply-chain cooldown:** Dependabot bump PRs are held for a detection window
  (pip patch/minor 5 days, major 10 days; GitHub Actions 5 days) so a
  compromised or yanked upstream release ages before adoption.
- **Pinned dependencies:** the Python toolchain is hash-pinned
  (`pip-compile --generate-hashes`, installed with `--require-hashes`), and all
  GitHub Actions are pinned to full commit SHAs rather than tags.
- **Vulnerability scanning:** Trivy filesystem scans run in CI on every push and
  pull request, with results published as SARIF to GitHub code scanning.
- **Static analysis:** CodeQL analyzes the GitHub Actions workflows for script
  injection, insecure action usage, and over-broad token permissions.
- **Least-privilege CI:** the default `GITHUB_TOKEN` is read-only; individual
  jobs grant only the additional scopes they require.
- **Vulnerability alerts:** GitHub Dependabot alerts and automated security
  updates are enabled.
- **Branch hygiene:** delete-branch-on-merge, conventional commit messages, and
  review via pull request.

## Role-Specific Security Considerations

Operators deploying this role should be aware of the following:

- **GPG verification is enabled by default.** Every repository this role defines
  sets `gpgcheck: true` with an explicit `gpgkey` path. Disabling GPG checking
  removes the only integrity control on packages served by the mirror. Do not
  override this.
- **`web_protocol` defaults to `https`.** Overriding it to `http` sends
  repository metadata and packages in cleartext. GPG checking still protects
  package integrity, but the transport provides no confidentiality and no
  protection against an on-path observer profiling the host's patch level.
- **`local_repo_server` must point at infrastructure you control.** The default
  is a non-routable placeholder. Pointing this at an untrusted or externally
  registrable hostname redirects all package installation for the target host.
- **This role removes vendor repository definitions**, including
  `/etc/yum.repos.d/redhat.repo` and the Oracle Linux repo files, and disables
  the Red Hat subscription-manager plugin. Confirm the local mirror is populated
  and reachable before applying the role, or the host will have no package
  source.

## Dependencies and Third-Party Code

This role depends on third-party Python packages used for linting and testing in
CI. It has no runtime dependencies beyond `ansible-core` and the
`ansible.builtin` module set. Vulnerabilities in those dependencies are
addressed by updating the relevant dependency to a patched version. If no
patched version exists, the maintainer will document the risk and apply
appropriate mitigations.
