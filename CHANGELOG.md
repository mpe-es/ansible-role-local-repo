# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed

- **BREAKING:** every role variable now carries the `local_repo_` prefix, so
  the interface is consistent and ansible-lint's `var-naming[no-role-prefix]`
  passes for roles that include this one. Renamed:
  `local_baseos_repo_folder` -> `local_repo_baseos_folder`,
  `local_appstream_repo_folder` -> `local_repo_appstream_folder`,
  `oracle9_baseos_repo_folder` -> `local_repo_oracle9_baseos_folder`,
  `oracle9_appstream_repo_folder` -> `local_repo_oracle9_appstream_folder`,
  `oracle9_gpg_key` -> `local_repo_oracle9_gpg_key`,
  `web_protocol` -> `local_repo_web_protocol`,
  `configure_local_cert_authority` -> `local_repo_configure_cert_authority`,
  `enclave_ca_certificates` -> `local_repo_enclave_ca_certificates`.
  The role does not detect the 1.x names: a variable left under its old name
  is ignored and the new name's default applies. A CA list left under
  `enclave_ca_certificates` with `local_repo_configure_cert_authority: true`
  installs the default `local_repo_enclave_ca_certificates` entry, which has
  no checksum pin, instead of the caller's list. The folder defaults are
  RHEL 10 paths, so an EL 8 or EL 9 host left on the 1.x folder names is
  pointed at RHEL 10 repositories without an error. See README "Upgrading from
  1.x". (#44)
- `meta/main.yml` now declares `min_ansible_version: '2.19'`, matching
  `system_base`, the role that includes this one. CI runs ansible-core 2.21
  only; `min_ansible_version` is Galaxy metadata, not enforced. EL 8 stays listed: the
  role uses no `dnf` module, so it runs on EL 8 hosts whose
  `ansible_python_interpreter` is a newer AppStream Python. See README
  "Requirements".
- The ansible-lint pre-commit hook is now a `repo: local` hook that runs the
  ansible-lint and ansible-core installed from `requirements.txt`. The hook
  previously pinned its own copies of both versions, which Dependabot does not
  update, so they drifted behind the lock and local lint disagreed with CI.
  The hook runs on every commit, as before. The config now requires pre-commit
  4.5.0 or later (`minimum_pre_commit_version`). Contributors must install
  `requirements.txt`, activate the venv if used, and run `pre-commit install`
  from that toolchain. See `CONTRIBUTING.md`. (#39)
- Every pre-commit hook now runs from the hash-pinned lock. The yamllint hook
  and the `pre-commit-hooks` hygiene hooks (trailing-whitespace,
  end-of-file-fixer, check-yaml, check-added-large-files,
  check-merge-conflict, detect-private-key) are `repo: local` hooks like
  ansible-lint, and `pre-commit-hooks` is pinned in `requirements.in`.
  pre-commit downloads nothing, Dependabot's pip updates cover every hook, and
  the yamllint hook no longer lags the lock (it was v1.35.1 against 1.38.0).
  (#40)
- The CI lint job runs `pre-commit run --all-files` in place of separate
  yamllint and ansible-lint steps, so CI also enforces the hygiene hooks and
  local and CI lint have one definition.
- `check-added-large-files` runs with `--enforce-all` and
  `check-merge-conflict` with `--assume-in-merge`. Without them both hooks
  only inspect staged additions and in-progress merges, so they passed every
  file in CI. They now reject any file over 500 KB and any committed conflict
  marker (staged files at commit time, every file in CI).
- Aligned the pre-commit hook's ansible-core with the lock at 2.21.6 (#27);
  superseded by the local-hook change above.

### Fixed

- Removed the extra trailing blank line from `tests/inventory`, which the
  end-of-file-fixer hook now enforces in CI.

### Security

- Bump the toolchain's transitive `filelock` from 3.32.3 to 4.2.10, fixing
  CVE-2026-112940 (HIGH) and the 11 other Dependabot alerts against it.
  `filelock` is only in the lint and CI toolchain (`requirements.txt`);
  nothing the role installs on a managed host was affected. (#51)

### Dependencies

- Bump pre-commit from 4.6.2 to 4.7.0 (#28)
- Bump ansible-lint from 26.9.0 to 26.10.1 (#29)
- Bump the codeql-action group with 3 updates (#30, #38)
- Bump virtualenv from 21.7.13 to 21.14.13 (#32)
- Bump actions/setup-python from 7.0.0 to 7.1.0 (#33)
- Bump black from 26.5.1 to 26.10.1 (#35)
- Bump reviewdog/action-actionlint from 1.79.1 to 1.81.0 (#36)
- Bump ansible-core from 2.21.4 to 2.21.6 (#37)
- Bump ansible-lint from 26.10.1 to 26.11.0 (#45)
- Bump reviewdog/action-actionlint from 1.81.0 to 1.84.0 (#46)
- Bump pre-commit-hooks from 5.0.0 to 6.0.0 (#47)

## [1.0.0] - 2026-10-15

### Added

- Optional enclave certificate authority trust, gated behind
  `configure_local_cert_authority` (default `false`, so existing consumers see
  no behavior change). The role installs each anchor listed in
  `enclave_ca_certificates` into `/etc/pki/ca-trust/source/anchors` and runs
  `update-ca-trust extract` before any repository file is written, because the
  mirrors are served over HTTPS by a host this CA issued. Anchors are fetched
  with `validate_certs: false` -- unavoidable during a trust bootstrap -- so
  each entry accepts an optional `checksum` pin, which is the control that
  substitutes for TLS validation and fails closed on mismatch.
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
- `CI Status` now requires `success` from every gated job rather than merely
  not-`failure`. `needs.<job>.result` also yields `cancelled` and `skipped`,
  and because the check carries `if: always()` it runs even when the workflow
  is cancelled, so a cancelled Security Scan previously passed the gate with
  no completed Trivy or SARIF evidence. (#10)
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

[Unreleased]: https://github.com/mpe-es/ansible-role-local-repo/compare/1.0.0...HEAD
[1.0.0]: https://github.com/mpe-es/ansible-role-local-repo/tree/1.0.0
