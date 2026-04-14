# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in split, **please do not open a public issue**.

Instead:

1. Email **balgaly@gmail.com** with a description of the vulnerability
2. Include steps to reproduce if possible
3. Allow reasonable time for a fix before any public disclosure

## Response Timeline

- **Acknowledgment**: Within 48 hours
- **Initial assessment**: Within 7 days
- **Fix**: Depends on severity — critical issues are prioritized

## Scope

This policy covers the split skill shell script and its ffmpeg integration.

## Security Practices

- Input validation on file paths and video extensions
- No silent sudo — ffmpeg installation requires explicit user permission
- No network access
- No telemetry or data collection

## Thank You

Security reports are taken seriously. Contributors who responsibly disclose vulnerabilities will be credited (unless they prefer to remain anonymous).
