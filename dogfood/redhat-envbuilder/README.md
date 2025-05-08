# FIPS-Compliant Red Hat envbuilder template for Coder 2.20.3

This template creates a Red Hat Enterprise Linux (RHEL) compatible development environment using the [envbuilder](https://github.com/coder/envbuilder) tool. It's based on the Red Hat Universal Base Image (UBI 9) which provides a RHEL-compatible environment that meets Federal Information Processing Standards (FIPS) requirements for high-security enterprise environments. This template is optimized for Coder v2.20.3.

## Features

- Built on Red Hat Universal Base Image (UBI 9)
- Compatible with Coder v2.20.3
- FIPS 140-2/140-3 compliant security configuration
- Validated cryptographic modules and libraries
- FIPS-enforced OpenSSL, Node.js, and Python configurations
- SELinux enabled in enforcing mode for security compliance
- Latest development toolchain with:
  - Go 1.24.3 (FIPS-validated)
  - Node.js 20.19.0 (LTS, FIPS-configured)
  - Terraform 1.11.5
  - Docker with latest components
- Integration with updated Coder modules for IDE support and productivity
- Systemd service management for proper service initialization
- Verified checksums for all downloaded software components

## FIPS Compliance Details

- Enforces the use of FIPS 140-2/140-3 validated cryptographic modules
- Restricts cryptographic algorithms to NIST-approved algorithms
- Configures OpenSSL in FIPS mode with proper validation
- Forces Node.js to use FIPS-compliant OpenSSL configuration 
- Verifies integrity of all downloaded packages with SHA-256 checksums
- Maintains SELinux in enforcing mode for system integrity

## Usage

1. Create a new workspace using this template
2. The template will build a Red Hat compatible container using the devcontainer.json
3. Connect to your workspace with your preferred IDE (VS Code, JetBrains Gateway)

## Customization

The startup script runs your `~/personalize` file if it exists, allowing you to customize your environment further.
Your home directory under `/home/coder` is persisted as a Docker volume, preserving your settings between workspace restarts.

## Parameters

- **Devcontainer Repository**: Git repository containing the devcontainer.json
- **Devcontainer Directory**: Directory within the repository containing the devcontainer.json
- **Region**: Geographic region for hosting your workspace

## Security Features

- FIPS 140-2/140-3 compliance fully enabled and enforced
- Cryptographic module validation at startup
- SELinux set to enforcing mode for mandatory access control
- SHA-256 checksum verification for all downloads
- Restricted cryptographic algorithms to NIST-approved only
- Red Hat security updates and vulnerability patching
- Enterprise-grade security policies with proper audit logging

## Enterprise Requirements

This template is specifically designed for environments with Red Hat Enterprise Linux requirements, providing a compatible development environment that meets enterprise security and compliance standards while still enabling modern development workflows.

## Known Issues

- Some tools may require additional configuration to work with SELinux enabled
- If you encounter permissions issues, you may need to adjust SELinux contexts

For troubleshooting or questions, please reach out to the DevOps team or the template maintainer.