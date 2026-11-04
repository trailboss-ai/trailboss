# Security policy

Trailboss reads sensitive data: agent transcripts (code, secrets, personal data) and the OAuth tokens agents store for plan-quota checks. We take reports seriously.

## Supported versions

Trailboss is pre-1.0. Only the latest release, and `main`, get security fixes.

## Reporting a vulnerability

**Don't open a public issue.** Report privately through the repo's **Security → Report a vulnerability** (a private GitHub security advisory).

Include what's affected, how to reproduce it, and the impact. Please don't include real transcripts or tokens.

You'll get an acknowledgement within 3 business days and an assessment within 10. Fixes are released as soon as they're ready, and reporters are credited in the advisory unless they'd rather not be.

## In scope

- Transcript content or credentials leaking: logs, error messages, files Trailboss writes, network traffic.
- Trailboss writing to, or corrupting, another agent's data.
- Command injection through text Trailboss types into terminals (session IDs, folder names, flags).
- Unsafe handling of files Trailboss reads (path traversal, symlink tricks).

## Out of scope

- Bugs in the agent CLIs themselves, or in Herdr or tmux. Report those upstream.
- Attacks that need an attacker who already controls your user account.
