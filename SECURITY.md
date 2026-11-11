# Security

This public repository contains documentation and sanitised sample output only. It
contains no executable code, credentials or tenant-specific data.

## Reporting a concern

If you find anything in this repository that should not be public — a credential, a
tenant identifier, an account ID, a client name, an internal reference — please do
**not** open a public issue. Email **<<YOUR CONTACT EMAIL>>** with the file path and
line. It will be removed and the history rewritten within one working day.

## Security of the tooling itself

The full tooling (private repository) handles Prisma Cloud and Cortex Cloud API
credentials. For licensed users, the private repository's `SECURITY.md` describes:

- credential storage (per-tenant config files outside the repository, never in
  arguments or environment dumps)
- API scopes required per subcommand, and why
- the release signing process for distributed binaries
- how to report a vulnerability in the tooling and the disclosure timeline

Defects in the tooling that affect a licensed engagement are handled through the
engagement's named contact.
