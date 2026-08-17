<!--
  Thank you for contributing to ansible-role-local-repo.
  Please complete the sections below. See CONTRIBUTING.md for full guidance.
-->

## Summary

<!-- What does this change do, and why? -->

## Type of change

- [ ] Bug fix (non-breaking change that fixes an issue)
- [ ] New platform or distribution support
- [ ] Enhancement to existing behavior
- [ ] Change to a default value in `defaults/main.yml`
- [ ] CI / tooling / supply-chain
- [ ] Documentation
- [ ] Breaking change (existing behavior changes for consumers)

## Related issues

<!-- e.g. "Closes #123". Use "Refs #123" for partial work. -->

## Testing performed

<!--
  Paste the result of the three local checks, and describe any manual
  verification against a real or test repository mirror.
-->

```
yamllint .
ansible-lint
ansible-playbook tests/test.yml -i tests/inventory --syntax-check
```

## Checklist

- [ ] Commits follow Conventional Commits (`feat:`, `fix:`, `docs:`, `ci:`, `deps:`, `chore:`)
- [ ] `yamllint` passes locally
- [ ] `ansible-lint` passes locally (profile `production`, `strict: true`)
- [ ] Playbook `--syntax-check` passes
- [ ] Every new or modified YAML file carries the UNCLASSIFIED classification banner
- [ ] `Last Updated` is current in the banner of every file I changed
- [ ] All modules use FQCN (`ansible.builtin.*`)
- [ ] Documentation (README / CONTRIBUTING / defaults) updated as needed

## Security checklist

<!-- This role controls where packages come from and how they are verified. -->

- [ ] `gpgcheck` remains `true` on every repository this role defines
- [ ] Every repository definition still has an explicit `gpgkey`
- [ ] `web_protocol` still defaults to `https`
- [ ] No default uses a publicly registrable hostname (use `.local` or `.invalid`)
- [ ] No secrets, credentials, internal hostnames, enclave names, or unit identifiers are included in the code, comments, or commit messages
- [ ] This PR does not report a security vulnerability (use private reporting instead)

<!--
  Do not report security vulnerabilities in a public pull request.
  See SECURITY.md for private reporting.
-->
