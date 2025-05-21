# 🔐 Vault Dynamic Secrets POC with HCP Vault & Render PostgreSQL

This is a proof-of-concept (PoC) to demonstrate how **HashiCorp Vault Dynamic Secrets** can be used with a **managed PostgreSQL database** hosted on [Render.com](https://render.com/). The PoC uses **HCP Vault** to dynamically generate short-lived database credentials, showcasing secure access without hardcoding static credentials.

## 🧩 Components

- **Vault (HCP Vault)**: Secure secret management and dynamic secret generation.
- **PostgreSQL (Render)**: Managed database service.
- **Vault Agent / CLI**: For interacting with Vault and generating credentials.

---

## 🚀 What You'll Learn

- How to configure Vault to connect to a PostgreSQL database.
- How to define roles and policies for dynamic secrets.
- How to retrieve time-bound database credentials securely.

---

## 📦 Prerequisites

- A running **HCP Vault** instance.
- A **PostgreSQL** instance on **Render**.
- `vault` CLI installed.
- Access to Vault via a **token** with sufficient privileges.

---

## 1️⃣ Configure Vault Postgres Database Secrets Engine

### Step 1: Enable the database secrets engine

```bash
vault secrets enable database
```
2️⃣ Create a Dynamic Role
This role tells Vault how to create temporary database users with specific privileges.

```bash
vault write database/roles/readonly-role \
    db_name=render-postgres \
    creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT CONNECT ON DATABASE <DB_NAME> TO \"{{name}}\";" \
    default_ttl="1h" \
    max_ttl="24h"
```bash

3️⃣ Vault Policy to Access Dynamic Credentials
Create a Vault policy to allow an application or user to retrieve credentials:
```bash
# policy.hcl
path "database/creds/readonly-role" {
  capabilities = ["read"]
}
```

Apply the policy:

```bash
vault policy write db-readonly policy.hcl
```

4.4️⃣ Fetch Dynamic Credentials
Use the token to fetch dynamic DB credentials:
```bash
VAULT_TOKEN=<TOKEN> vault read database/creds/readonly-role

```
sample output
```
Key                Value
---                -----
lease_id           database/creds/readonly-role/hN7DZz...
lease_duration     1h
lease_renewable    true
password           A1b2C3d4E5f6
username           v-token-readonly-role-xYZ

✅ Test DB Access
```

🔒 Benefits of Dynamic Secrets
Time-bound: Auto-expire after TTL.

Least Privilege: Each role has limited permissions.

No Secret Rotation Needed: Vault auto-generates credentials on demand.

Auditability: Every credential issue is logged.

