# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a single Bash script (`wireguard-install.sh`) that installs and manages a WireGuard VPN server on Linux. It supports interactive setup (first run) and a management menu (subsequent runs) for adding/listing/revoking clients and uninstalling.

## Supported Distributions

The script supports: Debian, Ubuntu, Fedora, CentOS (8+), AlmaLinux, Rocky Linux, Alibaba Cloud Linux, Amazon Linux 2, Oracle Linux, Arch Linux, and Alpine Linux.

## Lint Commands

```bash
# Run shellcheck (ignore SC1091, SC1117, SC2001, SC2034)
shellcheck -e SC1091,SC1117,SC2001,SC2034 wireguard-install.sh

# Run shfmt (diff mode, no in-place edit)
shfmt -d wireguard-install.sh
```

Both tools run on CI via GitHub Actions (`.github/workflows/lint.yml`).

## Code Structure

- `wireguard-install.sh` — Single file, ~600 lines. Functions are organized:
  - `initialCheck` → runs `isRoot`, `checkOS`, `checkVirt`
  - `installWireGuard` → runs `installQuestions`, installs packages per OS, generates keys, writes configs, starts service
  - `newClient` → generates client config and key pair, appends peer to server config
  - `listClients` / `revokeClient` → client management
  - `uninstallWg` → stops service, removes packages, cleans up config
  - `manageMenu` → displayed when WireGuard is already installed

- OS-specific logic uses the `OS` variable (from `/etc/os-release`) to branch package installation/uninstallation per distribution.

## Line Endings

The repository uses `.gitattributes` to ensure `*.sh` files always have LF line endings. Do not commit files with CRLF line endings.

## Contributing Guidelines

- Discuss changes in a GitHub issue before submitting a PR, especially for large changes
- Keep the script simple and easy to use
- Avoid large PRs; changes not aligned with project simplicity may be rejected