# Tycho

Tycho is an autonomous server management CLI designed to deploy and manage a complete, self-hosted infrastructure using Podman. It supports two primary deployment patterns.

For comprehensive setup guides and architectural instructions, check the [Tycho HOWTO](HOWTO.md).

## Key Features

- **Standalone CLI**: One tool to rule them all.
- **Dynamic Recipes**: Fetches the latest configurations directly from GitHub.
- **Rootless First**: Optimized for rootless Podman environments with strict user isolation.
- **Third-Party Repositories**: Helm-like custom recipe repositories allowing you to add, list, and deploy recipes from your own GitHub repositories, complete with automatic semantic tag resolution. (See [Repository User Guide](HOWTO.md#3-managing-repositories-user-guide) and [Recipe Provider Guide](HOWTO.md#4-being-a-recipe-provider-provider-guide)).
- **Automated Storage**: Each user gets their own isolated storage space (e.g., `/data/users/username/app`).
- **Smart Routing**: Automatic Traefik detection and subdomain conflict checks.

## Quick Install

```bash
curl -fsSL https://tycho.cc/install.sh | bash
```

---

## 🏗 Choose your Architecture

### 1. Personal Server (Full User Mode)
The easiest way for a single user. Everything (including Traefik) runs in your user space.
- **Prerequisite**: Allow binding to ports 80/443 (configured automatically by the interactive installer, or manually via):
  `echo "net.ipv4.ip_unprivileged_port_start=0" | sudo tee /etc/sysctl.d/99-rootless.conf && sudo sysctl --system`
- **Workflow**:
  ```bash
  tycho setup
  tycho install traefik    # Installs core/traefik in your home
  tycho install core/smtp
  tycho install immich     # Or any other recipe
  ```

### 2. Infrastructure Server (Shared/System Mode)
More robust. Traefik is a system-level gateway, and users install their apps separately.
- **Step A (Admin)**: Install the system gateway:
  ```bash
  sudo tycho setup
  sudo tycho install traefik  # Creates a systemd service
  ```
- **Step B (Users)**: Install personal apps:
  ```bash
  tycho setup
  tycho install immich
  ```

---

## Commands

- `tycho setup`: Interactive assistant to configure your domain and email.
- `tycho list`: Explore available recipes across all configured repositories.
- `tycho install [repo/]<pkg>`: Deploy a new service (interactive).
- `tycho uninstall <pkg>`: Cleanly remove a service and its data.
- `tycho repo <cmd>`: Manage third-party recipe repositories (add, list, remove, update, boilerplate). See the [Repository User Guide](HOWTO.md#3-managing-repositories-user-guide) for usage.
- `tycho stats`: Monitor your server's health.
- `tycho upgrade`: Self-update the CLI to the latest version.

## License

GNU AGPL-v3. See [LICENSE](LICENSE) for details.
