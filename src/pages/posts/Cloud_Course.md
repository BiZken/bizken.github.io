---
layout: ../../layouts/Post.astro
title: "Building a Cloud IoT Platform on Azure"
type: Project
description: "A walkthrough of the SIM600 Cloud Security labs — from an empty Azure subscription to a TLS-protected ThingsBoard instance with Azure AD single sign-on and secrets pulled from Key Vault at runtime."
tags: ["Azure", "Cloud Security", "IoT", "ThingsBoard", "OAuth2"]
backLink: /projects
backLabel: Back to Projects
---




# Building a Cloud IoT Platform on Azure: Three Labs, One System

*A walkthrough of the SIM600 Cloud Security labs — from an empty Azure subscription to a TLS-protected ThingsBoard instance with Azure AD single sign-on and secrets pulled from Key Vault at runtime.*

---

## Overlook

The labs consisted of three parts: one about Azure basics, one about setting up a server, one about certificates and OAuth. In practice they're the same system being built up layer by layer. By the end you have a working IoT data pipeline — a Raspberry Pi (or a Python simulator) publishing telemetry over MQTT, a ThingsBoard dashboard in a hardened Azure VNet, users authenticating through Azure AD, and an API that fetches its own secret from a Key Vault instead of hard-coding it.

---

## Lab 1 — Foundations: subscription, identities, and a first service

The first lab is less about Azure and more about the habits you want to establish before any real infrastructure exists.

**Group and subscription.** Three people, One person acts as Global Administrator and invites the others under `Subscription → Access control (IAM) → Add role assignment`. The role you hand out matters: the whole point of doing this with three people is that nobody should have access over everything (I did tho).

**Authentication methods.** Each person tries a different login method — Microsoft Authenticator, a TOTP app like Google Authenticator, ideally a FIDO2 key if you have one. Password-only is technically allowed for the lab but the point is to feel the difference between methods and internalize why MFA should be non-negotiable for cloud admin accounts.

**First services.** The group splits into three roles that map directly onto what you'd have in a real small company:

- **Person A** — Global Administrator, owns roles and rights.
- **Person B** — builds an Ubuntu VM (B-series, 2 GB, 30 GB standard SSD, auto-shutdown set).
- **Person C** — sets up a Storage Account with a `$web` container for a static site presenting the company.

The test at the end is the interesting part: Person B tries to delete Person C's storage account, and Person C tries to delete Person B's VM. **Neither should succeed.** If either works, the RBAC scoping is wrong and you start over. This is a concrete demonstration of least privilege — roles aren't just documentation, they physically prevent operations.

A few things that trip people up:

- The free tier requires starting from `azure.microsoft.com/free` with "Start free" — signing up a different way won't credit the 200 USD.
- Linux VMs require an SSH public key during creation; password-only auth on a public-facing VM is a bad default.
- `$web` can't be created manually. You enable **Static website** under the storage account's Data management, and Azure creates the container for you. The public URL lives under `Static website → Primary endpoint`, not on the overview page.

By the end of Lab 1 you haven't built anything impressive yet, but you have a subscription with proper identity hygiene, one VM, one static site, and a clear separation of who can do what.

---

## Lab 2 — The real infrastructure: VNet, split VMs, and ThingsBoard

Lab 2 is where the system starts looking like something a company would actually run. The goal is to host ThingsBoard — an open-source IoT platform — with a clean network topology.

### Network design

One **VNet** (`10.0.0.0/24`) with **two subnets**, and one VM in each:


The backend does not have a public IP is the single most important decision in this lab. The database is only reachable from inside the VNet, which means you cannot SSH into it directly from the internet — you SSH to the frontend, then hop to the backend over the private network. It's one of those "annoying until you're glad for it" patterns: an attacker who somehow finds your backend's public-facing port count will find zero, because there aren't any.

The frontend gets a DNS label through `Public IP → Configuration → DNS label`, giving you something like `company.northeurope.cloudapp.azure.com` instead of a raw IP. That matters a lot in Lab 3 when Let's Encrypt needs a real hostname to issue a certificate against.

### Installing the stack

The ThingsBoard install follows [their Ubuntu guide](https://thingsboard.io/docs/user-guide/install/ubuntu) with a split:

- **Java and ThingsBoard itself** go on the frontend only.
- **PostgreSQL** goes on the backend only.

The non-obvious work is in the middle — teaching the two to talk. Out of the box, PostgreSQL on Ubuntu only listens on `localhost`. For the frontend VM to connect, you have to:

1. Edit `postgresql.conf` and set `listen_addresses = '*'`.
2. Edit `pg_hba.conf` to add a host entry for the frontend's subnet using `md5` authentication.
3. Restart the service and verify with `ss -tlpn` that the daemon is now listening on `0.0.0.0:5432` instead of just `127.0.0.1:5432`.
4. Update ThingsBoard's config on the frontend to point at the backend's private IP with the Postgres credentials.

`ss -tlpn` is the diagnostic workhorse through this whole lab — it tells you exactly which process is listening on which interface, which is usually where things go wrong.

### Testing the pipeline

Once ThingsBoard is up, you create a device in the web UI, copy its access token, and paste it into the Python simulator (`sim-client.py`). The simulator publishes `temperature` and `shaking` telemetry over MQTT on port 1883 using the device token as the username. A few seconds later, values start appearing in the device's Telemetry tab.

That moment — watching sensor values stream into a dashboard you built from empty cloud — is the first time the labs feel like a product.

### The secrets count

The summary task at the end of Lab 2 asks you to count every credential you now need to manage. It's worth actually doing. At minimum:

- Azure account passwords + MFA (×3 group members)
- SSH private key(s) for the VMs
- `azureuser` login passwords
- PostgreSQL admin password
- ThingsBoard sysadmin password (default `sysadmin@thingsboard.org` / `sysadmin` — change it)
- ThingsBoard device access tokens (one per IoT device)

That's already 10+ secrets for a minimal two-VM deployment. This is the motivation for Lab 3: stop treating secrets like strings you paste into files.

---

## Lab 3 — Hardening: TLS, SSO, and secrets that aren't in your code

Lab 3 stacks three hardening layers on top of the working system: HTTPS everywhere, federated login through Azure AD, and secret management through Key Vault.

### Task A — Let's Encrypt TLS

ThingsBoard on port 8080 over plain HTTP means every login password is visible on the wire. The fix is certbot + Let's Encrypt:

```bash
sudo su -
apt update && apt install certbot
certbot certonly   # standalone mode spins up a temp web server on :80
```

Let's Encrypt's verification works via an HTTP-01 challenge: certbot opens port 80, Let's Encrypt fetches a specific file from your domain, and if it matches, your certificate gets signed. This is why you needed a real DNS label from Lab 2 — Let's Encrypt won't sign a raw IP or an internal hostname.

After a successful issue, the cert and key live under `/etc/letsencrypt/live/<domain>/`. ThingsBoard needs them copied into `/etc/thingsboard/conf/` with `thingsboard:thingsboard` ownership, and the service config needs four environment variables:

```bash
export SSL_ENABLED=true
export SSL_CREDENTIALS_TYPE=PEM
export SSL_PEM_CERT=fullchain.pem
export SSL_PEM_KEY=privkey.pem
```

Restart the service (this genuinely takes a couple of minutes — ThingsBoard is a heavy Java app), and HTTPS should come up. If it doesn't, `/var/log/thingsboard/` is where the answers live. A common miss is forgetting to `chown` the PEM files — ThingsBoard runs as its own user and will silently fail to read root-owned files.

The quick-fix copy-and-chown approach works but isn't ideal. In production you'd set up automatic renewal (Let's Encrypt certs expire in 90 days) and either have ThingsBoard read the live symlinks directly or wire a renewal hook that re-copies and restarts.

### Task B — OAuth2 single sign-on with Azure AD

This is the longest and most fiddly part of the lab, and the official ThingsBoard guides split the information across two pages — the [Azure-specific one](https://thingsboard.io/docs/user-guide/oauth/azure/) is old but explains the Azure-side app registration, and the [newer general one](https://thingsboard.io/docs/user-guide/oauth-2-support/) has the correct ThingsBoard-side configuration. You genuinely need to cross-reference both. For the new azure AD (entra ID) the documentation sucks. BTW

The flow, roughly:

1. User clicks "Login with Azure AD" in ThingsBoard.
2. ThingsBoard redirects the browser to `login.microsoftonline.com/<tenant>/oauth2/authorize`.
3. User signs in with their Azure AD account.
4. Azure AD redirects the browser back to ThingsBoard with an authorization code.
5. ThingsBoard exchanges the code for access and refresh tokens at `/oauth2/token`.
6. ThingsBoard validates the token against `discovery/keys` and either finds an existing tenant account or creates a new one from the Azure AD profile claims.

The Azure-side work is a standard app registration: register the app, add a redirect URI pointing at your ThingsBoard domain, generate a client secret, and note the tenant ID and client ID. On the ThingsBoard side you paste those into **System Settings → OAuth2**, along with the endpoint URLs derived from your tenant, the scopes (`openid profile email`), and a mapper configuration so Azure AD profile attributes get translated into ThingsBoard user fields.

The conceptual win here is that you've replaced a ThingsBoard-local username/password with a federated identity. If an employee leaves, you disable their Azure AD account once and they lose access to everything — ThingsBoard included. You've also gained MFA on the login for free, because the MFA policy is enforced by Azure AD, not by ThingsBoard.

### Task C — APIs and the Key Vault pattern

The last task is a miniature version of a real security improvement: stop storing secrets in source code.

The setup is an Azure Function App with a Python HTTP trigger — a tiny "hello, name" function protected by a function-level authorization code. Called from the frontend VM:

```python
import requests
secretvalue = "WZK4yx-Z2dVU6-ZUCAwL5wZM_ihhMF55INTpENZ0yAzFu-8JXgA=="
url = "https://hellotl.azurewebsites.net/api/hello?code="
r = requests.get(url + secretvalue + "&name=" + name)
```

That `secretvalue` is the problem. If this script ever gets pushed to GitHub — public or private — the secret is compromised. GitHub's secret scanning would catch a common-format token, but this one is a raw Azure function key and is much more likely to slip through. The Lab 2 homework literally asks you to research a historic breach caused by exactly this mistake (Uber's 2016 AWS keys in a GitHub repo is the classic example, but the list is long).

The Key Vault pattern fixes it:

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

credential = DefaultAzureCredential()
client = SecretClient(
    vault_url="https://VAULT_NAME.vault.azure.net/",
    credential=credential,
)
secret = client.get_secret("mysecret")
```

`DefaultAzureCredential` walks a chain of possible identity sources, and on an Azure VM it finds the **Managed Identity** assigned to that VM. The VM proves its identity to Azure through a metadata endpoint that's only reachable from inside the VM itself. No password, no token, no secret sitting on disk — the identity *is* the VM.

The first run will fail with something like `Forbidden — does not have secrets get permission`. You then:

1. Create a system-assigned managed identity on the frontend VM.
2. In the Key Vault, add an access policy (or RBAC assignment) giving that managed identity `Get` permission on secrets.

After that the VM can read the secret. No credentials live in the code, and anyone who steals the code from a laptop gets nothing — they can't impersonate the VM's managed identity. The final task of Lab 3 integrates the two programs: fetch the API token from the vault at startup, then loop calling the API with it.

---
