# Documenso on Unraid (Community Applications Template)

An opinionated guide for running **Documenso** (open‑source e‑signing) on **Unraid** using the CA template provided. It walks you through what each field means, safe defaults, how to keep your instance **private**, and how to run behind **Nginx Proxy Manager (NPM)**.

[![Template XML](https://img.shields.io/badge/Template-XML-blue)](https://raw.githubusercontent.com/Eurotimmy/unraid-templates/main/documenso/documenso.xml)
[![Icon](https://img.shields.io/badge/Icon-PNG-lightgrey)](https://raw.githubusercontent.com/Eurotimmy/unraid-templates/main/documenso/documenso.png)
[![Support Thread](https://img.shields.io/badge/Unraid%20Forum-Support-green)](https://forums.unraid.net/topic/193917-support-eurotimmy-documenso/)

> **Docs first**: If anything here conflicts with upstream, follow the official docs.  
> • Self‑hosting guide: https://docs.documenso.com/developers/self-hosting/how-to  
> • Docker env reference: https://raw.githubusercontent.com/documenso/documenso/main/docker/README.md  
> • Signing certificate: https://docs.documenso.com/developers/local-development/signing-certificate  
> • NextAuth (Auth.js v4) envs: https://next-auth.js.org/configuration/options

---

## 1) Where to get this template

- **Template XML:** `https://raw.githubusercontent.com/Eurotimmy/unraid-templates/main/documenso/documenso.xml`  
  Import this into Unraid CA (or use it as the Template URL in your custom template manager).  
- **Icon:** `https://raw.githubusercontent.com/Eurotimmy/unraid-templates/main/documenso/documenso.png`  
- **Support (Unraid Forum):** `https://forums.unraid.net/topic/193917-support-eurotimmy-documenso/`

---

## 2) Prerequisites

- Unraid server with CA plugin.
- A PostgreSQL database (local Docker or external).  
- SMTP credentials for email (used for invites, magic links, etc.).
- (Optional) S3/compatible storage (e.g., MinIO) if you prefer object storage over database file storage.
- (Recommended) A reverse proxy + TLS (e.g., Nginx Proxy Manager).

---

## 3) Quick Start (minimal working config)

Fill these **minimum** values in the Unraid template and start the container:

- **HTTP Port** → keep default `3000` unless you need to change the host port.
- **NEXT_PUBLIC_WEBAPP_URL** → `https://sign.example.com` (no trailing slash).  
- **NEXTAUTH_URL** → same as above, `https://sign.example.com` (no trailing slash).
- **AUTH_TRUST_HOST** → `true` (required when behind NPM or any reverse proxy).
- **NEXTAUTH_SECRET** → a long random string (see §8 Secrets & Keys).
- **NEXT_PRIVATE_ENCRYPTION_KEY** → a long random string (see §8 Secrets & Keys).
- **NEXT_PRIVATE_DATABASE_URL** → e.g. `postgresql://user:pass@dbhost:5432/documenso?sslmode=prefer`
- **NEXT_PRIVATE_DIRECT_DATABASE_URL** → same as above (direct/non‑pooled URL).
- **SMTP settings** → host, port, username, password, from name, from address.

> Start the container, visit your URL, and create the **first admin user**. Once set up, **disable public signups** (see §7).

---

## 4) Database settings (PostgreSQL)

Documenso expects standard Postgres URLs:

```
NEXT_PRIVATE_DATABASE_URL=postgresql://USER:PASS@HOST:5432/DBNAME?sslmode=prefer
NEXT_PRIVATE_DIRECT_DATABASE_URL=postgresql://USER:PASS@HOST:5432/DBNAME?sslmode=prefer
```

**Tips**
- Use separate DB user with least privileges.
- Prefer `sslmode=prefer` or `require` if your DB supports TLS.
- If you run Postgres as another Unraid Docker container, use the container name as the host and expose port 5432. Remember to persist Postgres data on a share.

---

## 5) Email (SMTP) settings

Set the following (transport defaults to `smtp-auth`):

```
NEXT_PRIVATE_SMTP_TRANSPORT=smtp-auth
NEXT_PRIVATE_SMTP_HOST=smtp.example.com
NEXT_PRIVATE_SMTP_PORT=587        # 587 STARTTLS or 465 SSL/TLS
NEXT_PRIVATE_SMTP_SECURE=false    # true if you use port 465
NEXT_PRIVATE_SMTP_USERNAME=<smtp-user>
NEXT_PRIVATE_SMTP_PASSWORD=<smtp-pass>
NEXT_PRIVATE_SMTP_FROM_NAME=Documenso
NEXT_PRIVATE_SMTP_FROM_ADDRESS=no-reply@example.com
```

**Hints**
- If using Gmail, create an **App Password** on an account with 2FA.
- Test deliverability (SPF/DKIM/DMARC); transactional emails often land in spam without correct DNS records.

---

## 6) File storage: `database` (default) vs `s3`

By default, Documenso stores files **in the database**:

```
NEXT_PUBLIC_UPLOAD_TRANSPORT=database
```

To use **S3 / MinIO** instead, set:

```
NEXT_PUBLIC_UPLOAD_TRANSPORT=s3
NEXT_PRIVATE_UPLOAD_ENDPOINT=https://s3.example.com        # e.g. http://minio:9000 or AWS S3 endpoint
NEXT_PRIVATE_UPLOAD_FORCE_PATH_STYLE=true                   # true for MinIO/compat endpoints
NEXT_PRIVATE_UPLOAD_REGION=us-east-1                        # region for S3
NEXT_PRIVATE_UPLOAD_BUCKET=documenso                        # pre-create the bucket
NEXT_PRIVATE_UPLOAD_ACCESS_KEY_ID=<your-access-key>
NEXT_PRIVATE_UPLOAD_SECRET_ACCESS_KEY=<your-secret-key>
```

**Notes**
- MinIO (self‑hosted) is a good choice on Unraid; expose it only inside your network or via a secure proxy.
- For AWS S3, ensure the IAM user has least‑privilege access to the target bucket.
- If switching from `database` to `s3` later, plan a migration path for existing files.

---

## 7) Make your instance **private** (disable public sign‑ups)

You have two options:

**A. Bootstrap then lock down (recommended):**
1. Start with `NEXT_PUBLIC_DISABLE_SIGNUP=false` (default).  
2. Create your admin account at `https://sign.example.com`.  
3. Stop the container, set `NEXT_PUBLIC_DISABLE_SIGNUP=true`, then start again.  
4. Invite additional staff users from inside Documenso.

**B. Keep private from the start:**
- Set `NEXT_PUBLIC_DISABLE_SIGNUP=true` before first run.
- Use SSO (OIDC) or invite flow (if available in your version) to add users.
- If you accidentally lock yourself out with no users, temporarily re‑enable sign‑ups, create the admin, then disable again.

**Extra hardening (NPM):**
- Add an **Access List** to allow only office IP ranges/users.
- Enable **HTTP Basic Auth** on NPM if you want a second login gate in front of Documenso.
- Ensure HTTPS is enforced.

---

## 8) Secrets & keys (safe generation)

**NEXTAUTH_SECRET** (Auth.js/NextAuth v4):  
Generate a 32‑byte (or longer) value. Examples:

```bash
# OpenSSL (base64)
openssl rand -base64 32

# URL-safe (Python 3)
python3 - <<'PY'
import secrets; print(secrets.token_urlsafe(48))
PY
```

**NEXT_PRIVATE_ENCRYPTION_KEY** (and optional secondary key):  
Use long, high‑entropy secrets (32+ chars). You can reuse the generation commands above. Keep them **private** and back them up securely.

**Signing certificate (.p12)** for document sealing:  
Follow the official guide to create or import a `.p12` and (optionally) set a passphrase:  
https://docs.documenso.com/developers/local-development/signing-certificate

Add to the template if you use one:

```
NEXT_PRIVATE_SIGNING_LOCAL_FILE_PATH=/opt/documenso/cert.p12
NEXT_PRIVATE_SIGNING_PASSPHRASE=<only-if-your-p12-has-one>
```

Mount your host file to `/opt/documenso/cert.p12` (Unraid “Path” mapping).

> **Keep all secrets out of screenshots and public repos.**

---

## 9) Running behind **Nginx Proxy Manager (NPM)**

1. **Add Proxy Host**
   - Domain: `sign.example.com`
   - Scheme: `http`
   - Forward Hostname/IP: the Unraid host IP or Docker container IP
   - Forward Port: **3000** (container port)
2. **SSL tab**
   - Request a certificate (Let’s Encrypt), enable **Force SSL** and **HTTP/2**.
3. **Advanced** (Custom Nginx config) – add:
   ```nginx
   proxy_set_header Host $host;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_set_header X-Forwarded-Proto $scheme;
   ```
4. **Websockets**: enable the **Websockets** toggle in NPM.
5. In the container env:
   - `NEXT_PUBLIC_WEBAPP_URL=https://sign.example.com`
   - `NEXTAUTH_URL=https://sign.example.com`
   - `AUTH_TRUST_HOST=true`

If you get auth callback errors, double‑check that both URLs exactly match your public HTTPS URL (no trailing slash).

---

## 10) Upload size & UX

- `NEXT_PUBLIC_DOCUMENT_SIZE_UPLOAD_LIMIT` defaults to `25` (MB). Set higher if needed.
- Very large PDFs or many signers can increase memory usage; size your container appropriately.

---

## 11) Updates & backups

- **App updates**: pull a newer image tag and restart. Review release notes first.
- **Back up**: Postgres database, any mounted certs (`.p12`), and (if using S3) ensure versioning or lifecycle policies.

---

## 12) Troubleshooting

- **502/Bad Gateway behind NPM** → likely missing Websockets toggle or wrong forward port.  
- **Auth loops** → `NEXTAUTH_URL`/`NEXT_PUBLIC_WEBAPP_URL` mismatch or missing `AUTH_TRUST_HOST=true`.  
- **Emails not sending** → incorrect SMTP, or provider requires SSL (`NEXT_PRIVATE_SMTP_SECURE=true` with port 465).  
- **Signing stuck** → verify your `.p12` is valid and the passphrase matches.  
- **Uploads fail with S3** → wrong endpoint/region/keys or missing bucket; with MinIO set `NEXT_PRIVATE_UPLOAD_FORCE_PATH_STYLE=true`.

---

## 13) Useful links (official)

- Self‑hosting Documenso: https://docs.documenso.com/developers/self-hosting/how-to  
- Docker env reference: https://raw.githubusercontent.com/documenso/documenso/main/docker/README.md  
- Signing certificate: https://docs.documenso.com/developers/local-development/signing-certificate  
- NextAuth envs: https://next-auth.js.org/configuration/options  
- NGINX reverse proxy concepts (for reference): https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/

---

## 14) Security checklist

- [ ] Public URL uses **HTTPS** and HSTS via NPM.  
- [ ] `NEXT_PUBLIC_DISABLE_SIGNUP=true` after creating the first admin.  
- [ ] Admin uses strong unique password + 2FA (if provider supports).  
- [ ] Database and S3 credentials are least‑privilege.  
- [ ] Secrets are long, random, backed up securely.  
- [ ] Backups tested (DB + certs + storage).  
- [ ] Access List and/or Basic Auth at NPM if extra privacy is desired.

---

If you improve this template, please contribute your changes back to the community via the Unraid forum thread or PRs to the template repo.

**Template XML:** https://raw.githubusercontent.com/Eurotimmy/unraid-templates/main/documenso/documenso.xml  
**Icon:** https://raw.githubusercontent.com/Eurotimmy/unraid-templates/main/documenso/documenso.png  
**Support:** https://forums.unraid.net/topic/193917-support-eurotimmy-documenso/
