[![GitHub release](https://img.shields.io/github/release/sgaunet/ansible-role-forgejo.svg)](https://github.com/sgaunet/ansible-role-forgejo/releases/latest)
[![CI](https://github.com/sgaunet/ansible-role-forgejo/workflows/CI/badge.svg)](https://github.com/sgaunet/ansible-role-forgejo/actions?query=workflow%3ACI)
[![License](https://img.shields.io/github/license/sgaunet/ansible-role-forgejo.svg)](LICENSE)

# Ansible Role: Forgejo

An Ansible Role that installs [Forgejo](https://forgejo.org/) binary on Linux systems.

Forgejo is a self-hosted lightweight software forge. Easy to install and low maintenance, it just does the job. Forgejo is a community-driven fork of Gitea.

This role implements the [binary installation method](https://forgejo.org/docs/latest/admin/installation/binary/) as described in the official Forgejo documentation, providing an automated and idempotent way to manage Forgejo binary installations across your infrastructure.

## Features

- Creates dedicated git user for Forgejo service
- Creates required data and configuration directories with proper permissions
- Downloads and installs Forgejo binary from official releases
- Generates complete `app.ini` configuration with security tokens
- Sets up systemd service with proper configuration and dependencies
- Version management with automatic upgrades/downgrades
- Idempotent installation (only downloads when version differs)
- Support for multiple Linux distributions
- Comprehensive molecule tests

## Requirements

- Ansible >= 2.15
- Linux system (required - Forgejo only provides Linux binaries)
- Internet connectivity to download Forgejo releases from Codeberg

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Target version of Forgejo to install
forgejo_target_version: "12.0.3"

# Installation directory for the forgejo binary
forgejo_install_path: "/usr/local/bin"
```

### Git User Configuration

The role creates a dedicated system user for Forgejo. You can customize these settings:

```yaml
# Git user configuration for Forgejo service
forgejo_user: "git"              # Username for Forgejo service
forgejo_group: "git"             # Group name for Forgejo service  
forgejo_home: "/home/git"        # Home directory for git user
forgejo_shell: "/bin/bash"       # Shell for git user
```

### Directory Configuration

The role creates the required directories for Forgejo data and configuration:

```yaml
# Forgejo directory configuration
forgejo_data_path: "/var/lib/forgejo"    # Data directory for repositories and application data
forgejo_config_path: "/etc/forgejo"      # Configuration directory for app.ini and other config files
```

### Systemd Service Configuration

The role automatically sets up a systemd service for Forgejo:

```yaml
# Systemd service configuration
forgejo_service_enabled: true                                    # Enable and start systemd service
forgejo_service_description: "Forgejo (Beyond coding. We forge.)"  # Service description
forgejo_service_restart_sec: "2s"                               # Restart delay after failure
forgejo_service_enable_capabilities: false                      # Enable CAP_NET_BIND_SERVICE for low ports
```

### Application Configuration

The role generates a complete `app.ini` configuration file with sensible defaults:

#### Database Configuration
```yaml
# Database configuration (SQLite is default)
forgejo_database_type: "sqlite3"                # Options: sqlite3, mysql, postgres
forgejo_database_host: "localhost"              # Database server host
forgejo_database_port: 3306                     # Database server port
forgejo_database_name: "forgejo"                # Database name
forgejo_database_user: "forgejo"                # Database user
forgejo_database_password: ""                   # Database password (use Ansible Vault)
```

#### Server Configuration
```yaml
# Server configuration
forgejo_server_protocol: "http"                 # Protocol: http or https
forgejo_server_domain: "localhost"              # Server domain
forgejo_server_http_port: 3000                  # HTTP port
forgejo_server_root_url: "http://localhost:3000/" # Full URL to Forgejo
forgejo_server_disable_ssh: false               # Disable SSH access
forgejo_server_ssh_port: 22                     # SSH port for Git operations
forgejo_server_ssh_listen_port: 2222            # Internal SSH listen port
```

#### Security & Service Configuration
```yaml
# Security (secrets auto-generated if empty)
forgejo_security_secret_key: ""                 # Secret key for encryption
forgejo_security_internal_token: ""             # Internal API token

# Service settings
forgejo_service_disable_registration: false      # Allow new user registration
forgejo_service_require_signin_view: false      # Require login to view content
forgejo_service_register_email_confirm: false   # Require email confirmation
```

#### Email Configuration
```yaml
# Mailer configuration
forgejo_mailer_enabled: false                   # Enable email notifications
forgejo_mailer_protocol: "sendmail"             # Protocol: sendmail or smtp
forgejo_mailer_smtp_addr: ""                    # SMTP server address
forgejo_mailer_smtp_port: 587                   # SMTP server port
forgejo_mailer_from: "Forgejo <noreply@localhost>" # From address
```

### Advanced Configuration

You can override the following variables for specific use cases:

```yaml
# Operating system (default: linux)
# Note: Currently only Linux binaries are provided by Forgejo
forgejo_os: "linux"

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

### Custom Installation Path

```yaml
---
- hosts: servers
  become: yes
  vars:
    forgejo_install_path: "/opt/forgejo/bin"
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

1. **User Setup**: Creates a dedicated `git` system user and group for Forgejo service
2. **Directory Creation**: Creates required directories with proper ownership and permissions:
   - Data directory: `/var/lib/forgejo` (git:git, 750)
   - Config directory: `/etc/forgejo` (root:git, 770)
3. **Version Check**: The role checks if Forgejo is already installed and gets its version
4. **Conditional Download**: Only downloads if the installed version differs from the target version
5. **Binary Installation**: Downloads the binary from Codeberg and installs it with proper ownership to the configured path (default: `/usr/local/bin`)
6. **Configuration**: Generates secure `app.ini` configuration file with auto-generated security tokens
7. **Systemd Service**: Creates and enables systemd service for automatic startup and management
8. **Verification**: Verifies the installation by running `forgejo -v`

## Supported Platforms

- **Enterprise Linux**: RHEL/CentOS/Rocky/AlmaLinux 8, 9
- **Ubuntu**: 20.04 (Focal), 22.04 (Jammy), 24.04 (Noble)
- **Debian**: 11 (Bullseye), 12 (Bookworm)
- **Fedora**: 38, 39, 40

**Note**: Forgejo currently only provides Linux binaries. The `forgejo_os` variable is available for future compatibility when other operating systems are supported.

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