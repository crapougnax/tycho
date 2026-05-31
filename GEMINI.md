# Tycho: Co-Development Principles

This document defines the foundational mandates, architectural principles, and strict code quality rules for the Tycho project. It serves as a guide for AI agents and human contributors to maintain consistency and integrity.

> [!IMPORTANT]
> **Global Directives**: This repository must also adhere to the user's personal coding preferences and customized rules:
> 👉 [Gist: Personal Gemini Rules & Instructions](https://gist.github.com/crapougnax/47971b85aa73dd702f4372a89858111c)

---

## 1. Architectural Mandates

- **Podman Centric**: All container management must use `podman`. Support for both rootless (User Mode) and rootful (System Mode) is mandatory.
- **Infrastructure Isolation**: 
    - **System Mode**: Infrastructure components like Traefik or VPNs should run as system-level services (Rootful/systemd) when managing host-level resources (ports 80/443).
    - **User Mode**: Applications must run in the user's rootless space.
- **Storage Separation**: Enforce strict isolation in shared environments. Data must be structured as `${BASE_STORAGE_PATH}/${USER}/${APP_NAME}` to allow for future quota management.

---

## 2. CLI Development & Code Quality Standards

The `tycho` CLI is a standalone, single-file Bash script designed to be fast, lightweight, and robust. To prevent bugs and ensure SonarCloud compliance, follow these strict rules:

### A. Prevent Exit Code Masking (Classic Bash Gotcha)
Never declare a `local` variable in the same statement as a command substitution (like `local output=$(curl ...)`). In Bash, `local` is itself a command that always returns exit status `0`, which masks the actual exit status of the command substitution.

* ❌ **BAD**:
  ```bash
  local tags_json=$(curl -s -f "...")
  if [[ $? -ne 0 ]]; then # $? will ALWAYS be 0, masking network errors!
  ```
* ✅ **GOOD**:
  ```bash
  local tags_json
  tags_json=$(curl -s -f "...")
  if [[ $? -ne 0 ]]; then # Correctly checks curl's exit status!
  ```

### B. Double Bracket Tests (`[[ ... ]]`)
Always use the double brackets construct `[[ ... ]]` instead of single brackets `[ ... ]` for conditional tests. The `[[ ... ]]` construct is safer, prevents word splitting/pathname expansion bugs, supports advanced pattern matching, and complies with SonarCloud static analysis rules (`shelldre:S7688`).

### C. Repository URL Sanitization
Always sanitize repository path arguments. If the user provides a full URL (e.g. `https://github.com/owner/repo` or `git@github.com:...`), the script must automatically strip the protocol, host, and optional `.git` suffix to extract the standard `owner/repo` path.

### D. Redirection to Stderr for CLI Sanity (`shelldre:S7677`)
Always redirect error messages, warnings, and diagnostic CLI headers to standard error (`>&2`). This preserves the integrity of stdout, allowing users to pipe command outputs cleanly.

### E. Mute JSON Parser Errors (`jq`)
When parsing remote outputs (which might be raw HTML, error texts, or empty payloads), always redirect `jq` errors to `/dev/null` (`2>/dev/null`) to prevent invalid JSON parse errors from cluttering the terminal output.

### F. Function Safeguards
- **Explicit Returns (`shelldre:S7682`)**: Always provide an explicit `return` statement (e.g., `return 0`) at the end of every shell function.
- **Merged Branching (`shelldre:S1066`)**: Avoid nested `if` statements when checking multiple environment conditions. Merge them into a single, cohesive check using logical operators (`&&` or `||`).

---

## 3. Recipe Standards

- **Metadata First**: Every recipe folder must contain a `package.json` with at least:
  - `name`: Unique identifier.
  - `description`: Short title for `tycho list`.
  - `requiredEnv`: List of mandatory environment variables.
- **Standardized Naming**: Use the format `${APPNAME}_SUBDOMAIN` for routing variables to prevent environment collisions.
- **Documentation**: A `README.md` is mandatory for each recipe to provide original project credits and specific usage instructions.

---

## 4. Security & Best Practices

- **Zero Secret Exposure**: Never hardcode keys or tokens. Use environment variables exclusively.
- **Least Privilege**: Favor rootless execution for all applications. Only use root for core network gateway components.
- **Persistence**: Always prefer persistent volumes over container storage. Ask for cleanup during uninstallation to prevent "ghost" data.

---

## 5. Development Workflow

- **Compatibility**: Any new feature must be tested for both single-user (home dir) and multi-user (shared `/data`) scenarios.
- **Versioning**: Maintain backward compatibility for recipes when updating the CLI logic.
- **Documentation**: Update the `HOWTO.md` for any new command or configuration logic.
