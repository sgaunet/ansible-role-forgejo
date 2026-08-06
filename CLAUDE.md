# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Ansible role that installs the Forgejo binary on Linux systems. Forgejo is a self-hosted lightweight software forge and a community-driven fork of Gitea.

## Key Commands

```bash
# Provision the toolchain: .venv (via uv) + Ansible collections
mise run deps

# Run linter (provisions deps first)
mise run lint

# Run molecule tests, requires Docker (provisions deps first)
mise run molecule-test

# The mise tasks above wrap these Taskfile targets
task lint
task tests

# List all available tasks
task

# mise puts .venv/bin on PATH, so the tools can be called directly
molecule test
molecule create
molecule converge
molecule verify
molecule idempotence
molecule destroy
ansible-lint .
```

## Architecture

### Role Structure
- **tasks/main.yml**: Main implementation that:
  1. Checks if Forgejo is installed and gets version using `{{ forgejo_install_path }}/forgejo -v`
  2. Downloads binary from Codeberg only if version differs from `forgejo_target_version`
  3. Installs to `{{ forgejo_install_path }}/forgejo`
  4. Verifies installation

### Key Variables (defaults/main.yml)
- `forgejo_target_version`: Version to install (default: "15.0.2")
- `forgejo_install_path`: Installation directory (default: "/usr/local/bin")
- `forgejo_os`: OS for binary (default: "linux" - only Linux binaries available currently)
- `forgejo_arch`: Architecture (default: "amd64", options: "amd64", "arm64")

### Forgejo Runner — split into a separate role
The Forgejo Runner (CI/CD) is no longer part of this role. It now lives in **`sgaunet.forgejo_runner`** (https://github.com/sgaunet/ansible-role-forgejo-runner). This role installs and configures the Forgejo **server** only.

### Testing Strategy
- Uses Molecule with Docker driver
- Tests run against Rocky Linux 9 container by default
- Can test other distros using: `MOLECULE_DISTRO=ubuntu2204 molecule test`
- Includes idempotence testing to ensure role doesn't make unnecessary changes

### Important Implementation Details

1. **Path Issues**: Always use full paths (`{{ forgejo_install_path }}/forgejo`) in commands since PATH may not be set correctly during Ansible execution

2. **Version Comparison**: The role compares exact version strings. The downloaded version has "v" prefix (e.g., "v15.0.2") which is stripped for comparison

3. **Conditional Logic**: Uses `("v" + forgejo_target_version)` syntax to avoid Jinja2 templating warnings in conditionals

4. **Download URL Format**: 
   ```
   https://codeberg.org/forgejo/forgejo/releases/download/v{{ forgejo_target_version }}/forgejo-{{ forgejo_target_version }}-{{ forgejo_os }}-{{ forgejo_arch }}
   ```

## Development Environment

- **mise**: Provisions the toolchain (Python 3.12, uv, task) and activates a project-local `.venv`; see `mise.toml`
- **uv**: Installs the pinned ansible-core/molecule/ansible-lint stack into `.venv`, so every tool shares one interpreter
- **Ansible collections**: Installed project-local into `.ansible/collections` via `ANSIBLE_COLLECTIONS_PATH`, not `~/.ansible`
- **Task**: Task runner for common commands
- **Molecule**: Testing framework using Docker containers
- **ansible-lint**: Code quality checks

## Galaxy Metadata (meta/main.yml)
- Role name: `sgaunet.forgejo`
- Minimum Ansible version: 2.15
- Supports: EL 8/9, Ubuntu (focal, jammy, noble), Debian (bullseye, bookworm), Fedora 38-40