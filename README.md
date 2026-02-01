# Auto Deploy

**Auto Deploy** is an Ansible-powered automation suite designed to manage and provision my entire hardware ecosystem. The primary goal is to minimize manual configuration and cognitive load when setting up development and testing environments.

---

## Infrastructure Tiers

My personal infrastructure is categorized into three distinct levels to streamline deployment strategies:

| Tier | Type | Description |
| :--- | :--- | :--- |
| **Level A** | **Private Homelab** | Local hardware, high-security, internal network. |
| **Level B** | **Public Machine** | VPS or cloud instances with public IP access. |
| **Level C** | **Non-Root Machine** | Shared environments where sudo/root access is unavailable. |

### Inventory Categories
Inventory files are named based on the machine's primary function:
* `dev`: Daily driver development instances.
* `stage`: Staging and pre-production environments.
* `other`: Specialized nodes (e.g., Mail servers, BGP routers).

---

## Environment Variables

The following variables must be configured (locally or in your CI/CD secrets) for the playbooks to execute successfully:

| Variable | Description | Used In |
| :--- | :--- | :--- |
| `DOTFILE_PRIVATE_KEY` | SSH key used to clone private dotfile repositories. | `bootstrap.yml`, `base-setup.yml` |
| `TARGET_USER` | The username to be created/configured on the remote host. | `bootstrap.yml` |
| `DEPLOY_KEY` | The SSH key used for CI/CD authentication. | CI/CD Runner |

---

## Playbooks

### 1. Bootstrap (`bootstrap.yml`)
Used for initial provisioning of a fresh machine.
* Creates the `TARGET_USER`.
* Deploys the public key to the target user's `authorized_keys`.
* **Requirement:** Must be run with root/sudo privileges.

### 2. Bootstrap Runner (`runner.yml`)
Configures a machine to act as an execution node (Jump Server).
* Syncs `.ssh` configurations from dotfiles.
* Maps SSH aliases to actual network addresses for seamless navigation.

### 3. Base Setup (`base-setup.yml`)
General configuration applied to **all** managed machines:
* Hardens SSH by disabling `PasswordAuthentication`.
* Installs essential CLI utilities and packages.
* Clones and links personal dotfiles.

### 4. Dev Setup (`dev-setup.yml`)
Tailored for development environments.
* Installs build toolchains (e.g., `ninja`, `cmake`, `gcc`).
* Configures language-specific runtimes.

---

## How to Add a New Machine

### For Tier A & B (Root Access)
Run the bootstrap playbook using a user with sudo privileges (often the default provider user):

```bash
TARGET_USER={target_user} ansible-playbook \
  -i {ip}, \
  -u {default_user} \
  --ask-pass --ask-become-pass \
  bootstrap.yml

### For Tier C (Non-Root)

Since root access is unavailable, manual intervention is required:

1. Manually append your public key to ~/.ssh/authorized_keys on the target machine.
2. Skip the bootstrap.yml playbook and proceed directly to base-setup.yml.
