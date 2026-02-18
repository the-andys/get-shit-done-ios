# Security Policy

## Reporting a Vulnerability

**Please do not report security vulnerabilities through public GitHub issues.**

### GSD Core Issues

For vulnerabilities in the core GSD framework (orchestration, agents, workflows):

- **Email:** security@gsd.build
- **Fallback:** DM @glittercowboy on Discord/Twitter

### iOS Fork Issues

For vulnerabilities specific to the iOS adaptations (Swift guidelines, iOS references, fork-specific changes):

- **GitHub:** Open a private security advisory on this repository
- **Email:** the-andys@users.noreply.github.com

### What to Include

- Description of the vulnerability
- Steps to reproduce
- Whether the issue is in GSD core or iOS-specific adaptations
- Potential impact
- Any suggested fixes (optional)

## Response Timeline

- **Acknowledgment**: Within 48 hours
- **Initial assessment**: Within 1 week
- **Fix timeline**: Depends on severity, but we aim for:
  - Critical: 24-48 hours
  - High: 1 week
  - Medium/Low: Next release

## Scope

Security issues in the GSD codebase that could:
- Execute arbitrary code on user machines
- Expose sensitive data (API keys, credentials, provisioning profiles, certificates)
- Compromise the integrity of generated plans/code

## Recognition

We appreciate responsible disclosure and will credit reporters in release notes (unless you prefer to remain anonymous).
