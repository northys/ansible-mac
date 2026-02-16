# CLAUDE.md

## Project Overview

This is a **macOS provisioning playbook** using Ansible. It automates the setup of a Mac development machine by installing packages (Homebrew, Homebrew Cask, npm) and deploying dotfiles/configuration. Originally authored for user "phillipalexander".

## Repository Structure

```
ansible-mac/
├── playbook.yml          # Main playbook (entry point, variables, task includes)
├── ansible.cfg           # Ansible configuration (local inventory, no host key checks)
├── hosts                 # Inventory file (localhost only, local connection)
├── README.md             # Brief usage instructions
├── tasks/                # Task files included by playbook.yml
│   ├── homebrew.yml      # Homebrew + Cask package installation
│   ├── npm.yml           # Global npm module installation
│   ├── files.yml         # Dotfile and config deployment (currently disabled)
│   ├── vim.yml           # spf13-vim installation (currently disabled)
│   └── zsh.yml           # Prezto/zsh setup (currently disabled)
└── files/                # Configuration files deployed to the target machine
    ├── gitconfig          # Git configuration (aliases, merge tools, colors)
    ├── gitignore          # Global gitignore rules
    ├── tmux.conf          # Tmux configuration (vi-style, powerline status)
    ├── slate.js           # Slate window manager key bindings
    ├── osx                # macOS `defaults write` customization script (~478 lines)
    ├── launchd.conf       # Global environment variable setup
    ├── jshintrc           # JSHint configuration
    └── Solarized-Dark-xterm-256color.terminal  # Terminal color scheme
```

## How to Run

```sh
# Install Ansible
brew install ansible

# Run the playbook (prompts for sudo password)
ansible-playbook -K ./playbook.yml

# Run specific tags only
ansible-playbook -K ./playbook.yml --tags homebrew
ansible-playbook -K ./playbook.yml --tags npm
```

## Architecture and Key Concepts

### Execution Model

- Runs **locally only** (`localhost ansible_connection=local` in `hosts`)
- Single playbook targeting `hosts: all`
- All variables are defined inline in `playbook.yml` under `vars:`
- No roles, no `group_vars/`, no `host_vars/`, no `requirements.yml`

### Active Task Files

| File | Tag | Purpose |
|------|-----|---------|
| `tasks/homebrew.yml` | `homebrew` | Updates Homebrew, installs CLI packages and Cask GUI apps |
| `tasks/npm.yml` | `npm` | Installs global npm modules |

### Disabled Task Files (commented out in playbook.yml)

| File | Tag | Purpose |
|------|-----|---------|
| `tasks/files.yml` | `files` | Deploys dotfiles from `files/` to home directory |
| `tasks/vim.yml` | `vim` | Installs spf13-vim via external script |
| `tasks/zsh.yml` | `zsh` | Clones Prezto fork and links runcom files |

### Variable Organization

All variables live in `playbook.yml` under `vars:`. Key variables:

- `user` - macOS username (hardcoded: `phillipalexander`)
- `home_dir` - User home directory path
- `homebrew_packages` - List of Homebrew CLI packages to install
- `homebrew_cask_packages` - List of Homebrew Cask GUI apps to install
- `npm_modules` - List of global npm packages to install

## Code Conventions

### Ansible Syntax (Legacy)

This project uses **pre-Ansible 2.0 syntax**. When making changes, maintain consistency with:

- **Module invocation**: `key=value` inline format, not YAML dict format
  ```yaml
  # This project's style:
  homebrew: name={{ item }} state=latest

  # NOT this:
  ansible.builtin.homebrew:
    name: "{{ item }}"
    state: latest
  ```

- **Loops**: `with_items:` (not `loop:`)
  ```yaml
  with_items: homebrew_packages
  ```

- **Privilege escalation**: `sudo: yes` (not `become: yes`)

- **Task includes**: `include:` with inline tags (not `import_tasks:` or `include_tasks:`)
  ```yaml
  - include: tasks/homebrew.yml tags=homebrew
  ```

### Naming

- **Variables**: `snake_case` (e.g., `homebrew_cask_packages`, `home_dir`)
- **Task names**: Sentence case descriptions (e.g., `"Update homebrew"`, `"Install npm packages"`)
- **Files in `files/`**: Named to match their dotfile counterpart without the dot (e.g., `gitconfig` deploys as `.gitconfig`)

### Disabling Features

Unused tasks are **commented out** in `playbook.yml` rather than removed:
```yaml
# - include: tasks/files.yml tags=files
```

## Important Notes for AI Assistants

1. **Personalization**: The playbook is configured for user `phillipalexander`. When suggesting changes, note that `user`, git email, and Prezto fork URL are personal values that need updating for other users.

2. **No tests or CI**: There is no test suite, linting, or CI/CD pipeline. Changes should be validated by reviewing Ansible syntax manually.

3. **Legacy syntax**: Do not modernize Ansible syntax unless explicitly asked. The `sudo:`, `with_items:`, and `key=value` module formats are intentional for this project's style.

4. **macOS-only**: This playbook targets macOS exclusively. Homebrew, Homebrew Cask, and `defaults write` commands are macOS-specific.

5. **No external dependencies**: There is no `requirements.yml`. All modules used are Ansible built-ins (`homebrew`, `homebrew_tap`, `homebrew_cask`, `npm`, `copy`, `git`, `file`, `command`, `shell`).

6. **Sensitive data**: The project references external secret files (`~/.secrets`, `.gitconfig.user`) that are not checked into the repository. Never add credentials or tokens to this repo.

7. **The `files/osx` script** is a standalone bash script (not called by any task) containing extensive macOS `defaults write` commands. It is run manually, not via Ansible.

## Common Modifications

### Adding a new Homebrew package

Add the package name to the `homebrew_packages` list in `playbook.yml`:
```yaml
homebrew_packages:
  - ack
  - hub
  - your-new-package  # add here
```

### Adding a new Cask application

Add the cask name to `homebrew_cask_packages` in `playbook.yml`:
```yaml
homebrew_cask_packages:
  - alfred
  - your-new-app  # add here
```

### Adding a new npm module

Add the module name to `npm_modules` in `playbook.yml`:
```yaml
npm_modules:
  - mocha
  - your-new-module  # add here
```

### Re-enabling a disabled task

Uncomment the relevant line in `playbook.yml`:
```yaml
- include: tasks/files.yml tags=files
```
