# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Ansible role that installs the Forgejo binary on Linux systems. Forgejo is a self-hosted lightweight software forge and a community-driven fork of Gitea.

## Key Commands

```bash
# Run tests (requires Docker)
task tests

# Run linter
task linter

# List all available tasks
task

# Run tests directly with molecule
devbox run -- molecule test

# Run specific molecule commands
devbox run -- molecule create
devbox run -- molecule converge
devbox run -- molecule verify
devbox run -- molecule idempotence
devbox run -- molecule destroy

# Run ansible-lint directly
devbox run -- ansible-lint .
```

## Architecture

### Role Structure
- **tasks/main.yml**: Main implementation that:
  1. Checks if Forgejo is installed and gets version using `{{ forgejo_install_path }}/forgejo -v`
  2. Downloads binary from Codeberg only if version differs from `forgejo_target_version`
  3. Installs to `{{ forgejo_install_path }}/forgejo`
  4. Verifies installation

### Key Variables (defaults/main.yml)
- `forgejo_target_version`: Version to install (default: "12.0.3")
- `forgejo_install_path`: Installation directory (default: "/usr/local/bin")
- `forgejo_os`: OS for binary (default: "linux" - only Linux binaries available currently)
- `forgejo_arch`: Architecture (default: "amd64", options: "amd64", "arm64")

### Testing Strategy
- Uses Molecule with Docker driver
- Tests run against Rocky Linux 9 container by default
- Can test other distros using: `MOLECULE_DISTRO=ubuntu2204 molecule test`
- Includes idempotence testing to ensure role doesn't make unnecessary changes

### Important Implementation Details

1. **Path Issues**: Always use full paths (`{{ forgejo_install_path }}/forgejo`) in commands since PATH may not be set correctly during Ansible execution

2. **Version Comparison**: The role compares exact version strings. The downloaded version has "v" prefix (e.g., "v12.0.3") which is stripped for comparison

3. **Conditional Logic**: Uses `("v" + forgejo_target_version)` syntax to avoid Jinja2 templating warnings in conditionals

4. **Download URL Format**: 
   ```
   https://codeberg.org/forgejo/forgejo/releases/download/v{{ forgejo_target_version }}/forgejo-{{ forgejo_target_version }}-{{ forgejo_os }}-{{ forgejo_arch }}
   ```

## Development Environment

- **Devbox**: Used for consistent development environment
- **Task**: Task runner for common commands
- **Molecule**: Testing framework using Docker containers
- **ansible-lint**: Code quality checks

## Galaxy Metadata (meta/main.yml)
- Role name: `sgaunet.forgejo`
- Minimum Ansible version: 2.15
- Supports: EL 8/9, Ubuntu (focal, jammy, noble), Debian (bullseye, bookworm), Fedora 38-40