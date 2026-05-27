# Tycho HOWTO

This guide provides detailed instructions on how to customize and extend your Tycho infrastructure depending on your needs.

## 1. Choice of Deployment Mode

Tycho allows you to choose between two architectural patterns.

### Personal Mode (Single User)
Everything (including Traefik) runs in your user space using Podman Rootless.
- **Location**: `~/.tycho/`
- **Installation**: Run all commands as a normal user.
- **Port Handling**: Requires permission to bind to ports 80/443 (handled by standard `sysctl net.ipv4.ip_unprivileged_port_start=0`).
- **Ideal for**: A simple private server where you are the only administrator.

### Infrastructure Mode (Shared Server)
Traefik runs at the system level (Rootful), while applications run in individual user spaces (Rootless).
- **Location**: `/etc/tycho/` (for system core) and `~/.tycho/` (for user apps).
- **Installation**: 
    1. Run `sudo tycho install traefik` to set up the system-wide gateway.
    2. Standard users run `tycho install <app>` to deploy their own services.
- **Port Handling**: Only `root` touches privileged ports.
- **Ideal for**: Servers shared by multiple users or more robust "enterprise-grade" home labs.

## 2. Configuration (.env)

The `.env` file is located at `~/.tycho/.env` (User) or `/etc/tycho/.env` (System).

- `DOMAIN_NAME`: Your base domain (e.g., `quatrain.dev`).
- `BASE_STORAGE_PATH`: The root directory where all apps will store their persistent data.
- `CF_DNS_API_TOKEN`: Cloudflare API token for wildcard SSL certificates.
- `TRAEFIK_RESOLVER`: `dnsresolver` (Cloudflare) or `myresolver` (HTTP).
- `TRAEFIK_AUTH`: Basic auth for the Traefik dashboard.

## 3. Managing Repositories (User Guide)

Tycho supports third-party recipe repositories similar to Helm. This allows you to add repositories hosted by the community or within your organization.

### Managing Repositories
- **Add a repository**:
  ```bash
  tycho repo add <name> <owner>/<repo>[@branch-or-tag]
  ```
  > [!TIP]
  > **Semantic Tag Resolution**: You can specify short version tags (e.g. `v1` or `v1.0`). Tycho will automatically query the GitHub API, resolve it to the latest matching full release tag (like `v1.0.4` or `v1.2.3`), and pin your configuration to that stable release.
  
  > [!WARNING]
  > **Security Warning**: Adding a repository without specifying a tag (e.g. `tycho repo add myrepo owner/repo`) is allowed and will default to the `main` branch. However, Tycho will print a security warning as this is not recommended for production stability.

- **List repositories**:
  ```bash
  tycho repo list
  ```
  Lists all added repositories alongside the default `official` one.

- **Verify repository status**:
  ```bash
  tycho repo update
  ```
  Loops through all configured repositories and verifies their online connectivity status.

- **Remove a repository**:
  ```bash
  tycho repo remove <name>
  ```

### Listing & Installing Third-Party Recipes
- **List recipes**: `tycho list` dynamically queries all registered repositories and presents a unified table:
  ```
  RECIPE                   REPOSITORY      DESCRIPTION
  --------------------------------------------------------------------------------
  nextcloud                official        Enterprise-grade sharing and collaboration platform
  quatrain-studio          coreapps        Visual data modeling and developer dashboard
  ```
- **Install a recipe**:
  - To install a recipe from any repository (searched automatically):
    ```bash
    tycho install quatrain-studio
    ```
  - To target a specific repository explicitly and avoid name collisions:
    ```bash
    tycho install coreapps/quatrain-studio
    ```

---

## 4. Being a Recipe Provider (Provider Guide)

If you are a developer or team lead, you can easily host your own Tycho recipe repository on GitHub to share custom services with your users.

### Repository Structure
Any GitHub repository can act as a Tycho recipe repository as long as it contains the designated `.tycho` root directory:
```
<your-repository-root>/
└── .tycho/
    ├── core/
    │   └── podman/                 # Core services (e.g., custom gateways)
    └── recipes/
        └── podman/                 # Platform recipes (e.g. podman, k8s coming soon)
            └── quatrain-studio/    # The recipe directory
                ├── compose.yaml    # Docker/Podman compose template
                ├── package.json    # Recipe metadata
                └── README.md       # Individual deployment instructions
```

### Recipe Components
Each recipe directory must include:
1. **`compose.yaml`** (or `compose.yml`): The Podman compose template. Route public HTTP services by declaring standard Traefik routing labels:
   ```yaml
   labels:
     - traefik.enable=true
     - traefik.http.routers.<app>.rule=Host(`${<APP>_SUBDOMAIN:-<app>}.${DOMAIN_NAME}`)
     - traefik.http.routers.<app>.tls=true
     - traefik.http.routers.<app>.entrypoints=websecure
     - traefik.http.routers.<app>.tls.certresolver=${TRAEFIK_RESOLVER:-myresolver}
     - traefik.http.services.<app>.loadbalancer.server.port=<port>
   ```
2. **`package.json`**: Metadata defining user query questions:
   ```json
   {
     "name": "quatrain-studio",
     "description": "Visual data modeling and developer dashboard",
     "requiredEnv": [
       "STUDIO_SUBDOMAIN",
       "STUDIO_DATA_LOCATION"
     ]
   }
   ```
3. **`README.md`**: Descriptive setup, customization options, and volume mount documentation for users.

### Best Practices for Recipe Providers
- **Semantic Version Tags**: Proactively tag your repository releases using semantic versioning (e.g., `v1.0.0`, `v1.1.2`, `v2.0.0`). This allows users to add your repository pinned to safe major or minor release versions (like `owner/repo@v1`), which Tycho resolves to the latest stable minor/patch update dynamically.

## 4. Monitoring & Maintenance

- **System Stats**: `tycho stats` shows real-time resource usage.
- **Process List**: `tycho ps` lists all active containers managed by Tycho for the current user.
- **Upgrade CLI**: `tycho upgrade` downloads the latest version of the management script from GitHub.
- **Uninstall**: `tycho uninstall <pkg>` stops the service and optionally cleans up volumes and networks.

## 5. Troubleshooting

- **Logs**: `tycho logs <package>` (e.g., `tycho logs core/traefik`).
- **Conflict detection**: Tycho will warn you if you try to use a subdomain that is already active on the same server.
