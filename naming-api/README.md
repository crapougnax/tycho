# Tycho Naming API

The **Tycho Naming API** is the server-side Dynamic DNS (DDNS) registration service for the Tycho ecosystem. It enables clients equipped with the Tycho CLI to request and dynamically update a level-2 wildcard subdomain (e.g., `*.toto.tycho.cc`) in a secure and privacy-respecting manner.

This service is built on top of the **Quatrain Core** stack (Express server and SQLite database adapter).

---

## 🌟 Features

- **Automatic Wildcard DNS:** Registers both `toto.tycho.cc` (A Record) and `*.toto.tycho.cc` (CNAME Record).
- **Absolute Pseudonymity (Zero-Knowledge):** The user's email address is hashed using `HMAC-SHA256` with a secret server salt. No plain-text identities are stored in the database.
- **Abuse Prevention & Security:**
  - Strict syntax validation and blocking of reserved keywords (`admin`, `api`, `secure`...).
  - Strict rejection of loopback and private IP addresses (RFC 1918) to prevent *DNS Rebinding* attacks.
  - Rate limiting and domain quota checking (maximum of 5 subdomains per user account).
- **Direct Ionos Integration:** Interacts directly with the Ionos developer DNS API to update your zone.

---

## 🛠️ Environment Variables

The service can be customized using the following environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `PORT` | The HTTP port the server listens on | `4000` |
| `API_SECRET_SALT` | Cryptographic secret salt used for hashing user emails pseudonymously | `default_tycho_secret_salt_123!` |
| `DOMAIN_SUFFIX` | The main zone/domain to manage (e.g., `tycho.cc`) | `tycho.cc` |
| `IONOS_API_PREFIX` | Public prefix of your Ionos developer API Key | *(Required for prod)* |
| `IONOS_API_SECRET` | Secret of your Ionos developer API Key | *(Required for prod)* |

*Note: If Ionos API credentials are not provided, the API runs in **simulation mode** (ideal for local testing).*

---

## 📦 Deployment

The service is fully containerized. To deploy it on your main server hosting `tycho.cc`:

1. Create a storage folder for the SQLite database:
   ```bash
   mkdir -p /data/naming-api
   ```

2. Spin up the container using Podman (or Docker):
   ```bash
   podman compose up -d
   ```

The service will automatically expose itself on your local Traefik gateway at the address `api.tycho.cc`.

---

## 📡 API Endpoints

### 1. Register / Update IP
- **Route:** `POST /v1/alias/register`
- **Headers:** `Content-Type: application/json`
- **Request Body:**
  ```json
  {
    "subdomain": "toto",
    "email": "user@email.com",
    "ip": "82.120.45.67"
  }
  ```
- **Responses:**
  - `200 OK`: Alias registered or IP updated successfully.
  - `400 Bad Request`: Private IP, invalid subdomain format, or reserved name.
  - `403 Forbidden`: Subdomain is already registered by another email account.
  - `429 Too Many Requests`: Subdomain quota limit (5) reached for this account.

### 2. Healthcheck
- **Route:** `GET /health`
- **Response:**
  ```json
  { "status": "ok", "domainSuffix": "tycho.cc" }
  ```
