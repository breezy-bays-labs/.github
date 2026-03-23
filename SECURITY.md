# Security Policy

This file is the default security policy for repositories in the
`breezy-bays-labs` organization.

If a repository has its own `SECURITY.md`, that repository-specific policy
takes precedence over this default.

## Reporting a Vulnerability

Do not open a public issue for a suspected security vulnerability.

Preferred reporting path:

- If the repository has GitHub private vulnerability reporting enabled, use the
  repository's **Security** tab and choose **Report a vulnerability**.

Fallback reporting path:

- If the repository does not have private vulnerability reporting enabled, email
  `breezy-bays-labs@proton.me`.

Include:

1. A description of the vulnerability
2. Steps to reproduce
3. Affected repositories, versions, or commits
4. Any potential impact assessment

We aim to acknowledge reports within 48 hours and provide a status update within
7 days.

## Scope

This policy covers confidential reports involving:

- Secrets exposure
- Dependency or supply-chain compromise
- Privilege escalation or auth bypass
- Unsafe file, network, or command execution paths
- Data exposure or tampering risks

Issues that are purely documentation defects, feature gaps, or normal usage
bugs should be reported through the repository's usual issue/support channels
instead.
