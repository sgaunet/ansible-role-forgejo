[![GitHub release](https://img.shields.io/github/release/sgaunet/ansible-role-forgejo.svg)](https://github.com/sgaunet/ansible-role-forgejo/releases/latest)
[![CI](https://github.com/sgaunet/ansible-role-forgejo/workflows/CI/badge.svg)](https://github.com/sgaunet/ansible-role-forgejo/actions?query=workflow%3ACI)
[![License](https://img.shields.io/github/license/sgaunet/ansible-role-forgejo.svg)](LICENSE)

# Ansible Role: Forgejo

An Ansible Role that installs [Forgejo](https://forgejo.org/) binary on Linux systems.

Forgejo is a self-hosted lightweight software forge. Easy to install and low maintenance, it just does the job. Forgejo is a community-driven fork of Gitea.

This role implements the [binary installation method](https://forgejo.org/docs/latest/admin/installation/binary/) as described in the official Forgejo documentation, providing an automated and idempotent way to manage Forgejo binary installations across your infrastructure.

## Features

- Downloads and installs Forgejo binary from official releases
- Version management with automatic upgrades/downgrades
- Idempotent installation (only downloads when version differs)
- Support for multiple Linux distributions
- Comprehensive molecule tests

## Requirements

- Ansible >= 2.15
- Linux system (EL 8/9, Ubuntu 20.04+, Debian 11+, Fedora 38+)
- Internet connectivity to download Forgejo releases from Codeberg

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Target version of Forgejo to install
forgejo_target_version: "12.0.3"
```

### Advanced Configuration

You can override the following variables for specific use cases:

```yaml
# Custom installation path (default: /usr/local/bin/forgejo)
forgejo_install_path: "/usr/local/bin/forgejo"

# Architecture (default: amd64, options: amd64, arm64)
forgejo_arch: "amd64"
```

## Dependencies

None.

## Example Playbook

### Basic Installation

```yaml
---
- hosts: servers
  become: yes
  roles:
    - sgaunet.forgejo
```

### Specific Version Installation

```yaml
---
- hosts: servers
  become: yes
  vars:
    forgejo_target_version: "12.0.3"
  roles:
    - sgaunet.forgejo
```

### Multiple Servers with Different Versions

```yaml
---
- hosts: production
  become: yes
  vars:
    forgejo_target_version: "12.0.3"
  roles:
    - sgaunet.forgejo

- hosts: staging
  become: yes
  vars:
    forgejo_target_version: "12.0.2"
  roles:
    - sgaunet.forgejo
```

## Installation

### Via Ansible Galaxy

```bash
ansible-galaxy install sgaunet.forgejo
```

### Via requirements.yml

Add this to your `requirements.yml`:

```yaml
---
roles:
  - name: sgaunet.forgejo
    version: "1.0.0"  # Check for latest version
```

Then install:

```bash
ansible-galaxy install -r requirements.yml
```

## Development

### Prerequisites

- [Devbox](https://www.jetify.com/devbox) for development environment
- [Docker](https://www.docker.com/) for molecule tests
- [Task](https://taskfile.dev/) for task automation

### Setup Development Environment

```bash
# Install devbox if not already installed
curl -fsSL https://get.jetify.com/devbox | bash

# Enter the development shell
devbox shell

# Or run commands directly
devbox run <command>
```

### Testing

This role is tested with [Molecule](https://molecule.readthedocs.io/) using Docker containers.

```bash
# Run all tests
task tests

# Or using devbox directly
devbox run -- molecule test

# Run specific test sequence
devbox run -- molecule create
devbox run -- molecule converge
devbox run -- molecule verify
devbox run -- molecule destroy

# Test idempotence
devbox run -- molecule converge
devbox run -- molecule idempotence
```

### Linting

```bash
# Run ansible-lint
task lint

# Or
devbox run -- ansible-lint
```

### Directory Structure

```
ansible-role-forgejo/
├── defaults/
│   └── main.yml           # Default variables
├── meta/
│   └── main.yml           # Role metadata for Ansible Galaxy
├── molecule/
│   └── default/
│       ├── converge.yml   # Playbook for molecule tests
│       ├── molecule.yml   # Molecule configuration
│       └── verify.yml     # Verification tests
├── tasks/
│   └── main.yml           # Main task file
├── .gitignore
├── devbox.json            # Devbox configuration
├── devbox.lock            # Devbox lock file
├── LICENSE                # MIT License
├── README.md              # This file
└── Taskfile.yml           # Task automation
```

## How It Works

1. **Version Check**: The role first checks if Forgejo is already installed and gets its version
2. **Conditional Download**: Only downloads if the installed version differs from the target version
3. **Binary Installation**: Downloads the binary from Codeberg and installs it to `/usr/local/bin`
4. **Verification**: Verifies the installation by running `forgejo -v`

## Supported Platforms

- **Enterprise Linux**: RHEL/CentOS/Rocky/AlmaLinux 8, 9
- **Ubuntu**: 20.04 (Focal), 22.04 (Jammy), 24.04 (Noble)
- **Debian**: 11 (Bullseye), 12 (Bookworm)
- **Fedora**: 38, 39, 40

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Development Guidelines

- Follow Ansible best practices
- Add molecule tests for new features
- Update documentation as needed
- Ensure all tests pass before submitting PR
- Use conventional commits format

## Troubleshooting

### Common Issues

1. **Permission Denied**: Ensure you're running the playbook with `become: yes` for system-wide installation

2. **Download Failures**: Check your internet connectivity and firewall rules for accessing Codeberg

3. **Version Not Changing**: The role compares exact version strings. Ensure `forgejo_target_version` matches the release version format (e.g., "12.0.3" not "v12.0.3")

## License

MIT

## Author Information

This role was created by [Sylvain Gaunet](https://github.com/sgaunet).

## Links

- [Forgejo Official Website](https://forgejo.org/)
- [Forgejo Documentation](https://forgejo.org/docs/latest/)
- [Forgejo Binary Installation Guide](https://forgejo.org/docs/latest/admin/installation/binary/)
- [Forgejo Releases](https://codeberg.org/forgejo/forgejo/releases)
- [Ansible Galaxy Page](https://galaxy.ansible.com/sgaunet/forgejo)