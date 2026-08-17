# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Community health files: `LICENSE` (Apache 2.0), `CODE_OF_CONDUCT.md`
  (Contributor Covenant 2.1), `CONTRIBUTING.md`, `SECURITY.md`, and
  `CODEOWNERS`.
- Issue forms for bug reports and feature requests, plus a pull request
  template. The bug form asks for the repository mirror state (server
  reachable, paths populated, GPG key present), which covers the most common
  causes of a failure that presents as a role defect but is not.
- `UNCLASSIFIED` classification banner on every YAML and configuration file.
  Coverage is now complete across the repository.
- Dependabot `codeql-action` lockstep group, so `github/codeql-action` `init`,
  `analyze`, and `upload-sarif` are proposed as one pull request at a single
  version instead of drifting apart into a broken analysis. (#4)
- GitHub topics matching the existing `galaxy_tags`, for discoverability.

### Changed

- Reconciled the declared license. `meta/main.yml` and `README.md` both stated
  MIT while the shipped `LICENSE` is Apache 2.0. Both declarations are now
  Apache-2.0 to match the file.
- Generalized task names to describe the repository mirror rather than a
  specific deployment or RHEL major version. One AppStream task claimed RHEL 8
  while the role defaults ship RHEL 10 paths.
- `local_repo_server` now defaults to a reserved `.local` hostname. The
  previous default sat under a publicly registrable TLD, so an unmodified run
  on a host with egress could resolve and reach a third-party server.
- Removed the optional `galaxy_info.company` field.
- Reworded the README summary from Yum to DNF.

### Fixed

- Granted `actions: read` to the `security` job. `github/codeql-action/upload-sarif`
  calls the workflow-run API to resolve the run it attaches results to, which
  requires that scope on a private repository. Without it the upload step
  failed with `Resource not accessible by integration`. The `analyze` job in
  `codeql.yml` already granted it; the two workflows are now consistent.

### Security

- `CI Status` now gates on the `security` job. The job was listed in the
  check's `needs` but omitted from its failure condition, so a Trivy CRITICAL
  or a CodeQL finding failed the Security Scan job while the single required
  status context for branch protection still reported success and the pull
  request stayed mergeable. (#9)
- Enabled Dependabot alerts, Dependabot security updates, the dependency
  graph, secret scanning, secret scanning push protection, private
  vulnerability reporting, and code scanning ingestion for both CodeQL and
  Trivy SARIF.
- Enabled branch protection on `main`: pull request required, `CI Status`
  required, force pushes and deletions blocked.
- Documented role-specific security considerations in `SECURITY.md`: GPG
  verification must stay enabled, `web_protocol` must not be downgraded to
  `http`, `local_repo_server` must point at infrastructure you control, and
  the role removes vendor repository definitions so the mirror must be
  populated before it is applied.

### Dependencies

- Bump ansible-core from 2.21.1 to 2.21.2
- Bump pre-commit from 4.6.0 to 4.6.1
- Bump ansible-lint from 26.6.0 to 26.8.0 (#1)
- Bump pre-commit from 4.6.1 to 4.6.2 (#2)
- Bump ansible-core from 2.21.2 to 2.21.3 (#3)
- Bump github/codeql-action from 4.37.0 to 4.37.6 (`init`, `analyze`,
  `upload-sarif` in lockstep) (#4)
- Bump reviewdog/action-actionlint from 1.72.0 to 1.73.1 (#5)
- Bump actions/setup-python from 6.3.0 to 7.0.0 (#6)
- Bump actions/checkout from 7.0.0 to 7.0.1 (#7)

[Unreleased]: https://github.com/mpe-es/ansible-role-local-repo/commits/main
