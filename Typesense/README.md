# Typesense (Unraid CA Template)

Typesense is an open-source, typo-tolerant search engine with fast full-text search, faceting/filtering, synonyms, geo-search and vector search. This Unraid template deploys the official `typesense/typesense:29.0` image, exposes the HTTP API on port **8108**, and persists data in `/data`.

---

## 1) Template fields (what to fill in)

- **WebUI / API Port**: `8108`
- **Data Location (Path)**: `/mnt/user/appdata/typesense/data` → container path `/data`
- **Environment**
  - `TYPESENSE_API_KEY` *(required)*: A long, random admin key. Keep it private.
  - `TYPESENSE_DATA_DIR` *(required)*: `/data`  ← **This fixes the “Invalid configuration: Data directory is not specified.” error.**
  - `TYPESENSE_ENABLE_CORS` *(optional)*: `true` if you’ll call the API from a browser app.

> Tip: Typesense supports config via **environment variables** that map directly to server flags (`--data-dir` → `TYPESENSE_DATA_DIR`, `--api-key` → `TYPESENSE_API_KEY`, etc.).

---

## 2) Quick health check

After starting the container:

```bash
# If running on the Unraid host:
curl -s http://localhost:8108/health
# Expected: {"ok":true}
```

All endpoints **except** `GET /health` require an API key sent as `X-TYPESENSE-API-KEY` (or `?x-typesense-api-key=`).

---

## 3) First steps: create → import → search

Set your key for convenience:

```bash
export TYPESENSE_URL="http://<YOUR-HOST>:8108"
export TYPESENSE_API_KEY="<YOUR-ADMIN-KEY>"
export H="X-TYPESENSE-API-KEY: $TYPESENSE_API_KEY"
```

### 3.1 Create a collection

```bash
curl -s -X POST "$TYPESENSE_URL/collections"   -H "$H" -H "Content-Type: application/json"   -d '{
    "name": "products",
    "fields": [
      {"name": "id",         "type": "string"},
      {"name": "title",      "type": "string"},
      {"name": "price",      "type": "float"},
      {"name": "categories", "type": "string[]"},
      {"name": "created_at", "type": "int64"},
      {"name": "embedding",  "type": "float[]", "optional": true}
    ],
    "default_sorting_field": "created_at"
  }'
```

### 3.2 Import a few documents (JSONL)

```bash
cat > products.jsonl <<'JSONL'
{"id":"1","title":"Steel Shed Alpha","price":19990,"categories":["shed","steel"],"created_at":1700000000}
{"id":"2","title":"Timber Shed Bravo","price":14990,"categories":["shed","timber"],"created_at":1700500000}
{"id":"3","title":"Rural Farm Shed","price":29990,"categories":["shed","rural"],"created_at":1701000000}
JSONL

curl -s --data-binary @products.jsonl   -H "$H" -H "Content-Type: text/plain"   "$TYPESENSE_URL/collections/products/documents/import?action=upsert&batch_size=100"
```

Typesense’s bulk import expects **JSONL** (one JSON object per line).

### 3.3 Search

```bash
# Note: quote the URL to keep & characters intact in your shell
curl -s -H "$H"   "$TYPESENSE_URL/collections/products/documents/search?q=shed&query_by=title,categories&filter_by=price:>=15000&&price:<=30000&per_page=5"
```

---

## 4) Create a search-only key (recommended for front-end apps)

```bash
curl -s -X POST "$TYPESENSE_URL/keys"   -H "$H" -H "Content-Type: application/json"   -d '{
    "description": "Search-only key for products",
    "actions": ["documents:search"],
    "collections": ["products"],
    "expires_at": 0
  }'
```

Use the returned key in the browser/client SDKs, not the admin key.

---

## 5) Backups (snapshots) & restore

Create a **point-in-time snapshot** to a folder inside your data volume, then back up that folder with your usual Unraid backup process:

```bash
# Ensure the snapshot target exists and is writable by the container
mkdir -p /mnt/user/appdata/typesense/data/snapshots

# Trigger a snapshot
curl -s -H "$H"   "$TYPESENSE_URL/operations/snapshot?snapshot_path=/data/snapshots/$(date +%F_%H%M%S)"
```

To restore: stop the container and replace `/mnt/user/appdata/typesense/data` with one of the snapshot directories (or repoint to it), then start Typesense again.

---

## 6) Simple Nginx reverse proxy

> If you’re using Nginx Proxy Manager, translate the bits below into the UI (Proxy Host → Forward Host `TYPESENSE_CONTAINER:8108`, and add the headers under “Advanced”).

```nginx
# /etc/nginx/conf.d/typesense.conf
server {
  listen 443 ssl http2;
  server_name search.example.com;

  # TLS config omitted for brevity

  location / {
    proxy_pass         http://127.0.0.1:8108;
    proxy_set_header   Host              $host;
    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;

    # Helpful timeouts for large imports
    proxy_read_timeout  600s;
    proxy_send_timeout  600s;
  }

  # (Optional) Lock down by IP for admin access
  # location / {
  #   allow 203.0.113.0/24;
  #   deny  all;
  #   proxy_pass http://127.0.0.1:8108;
  # }
}
```

If your UI calls Typesense directly from a browser, keep `TYPESENSE_ENABLE_CORS=true` on the server and/or add the appropriate CORS rules in your proxy layer.

---

## 7) Troubleshooting

- **`Invalid configuration: Data directory is not specified.`** → Ensure `TYPESENSE_DATA_DIR=/data` is present *and* your path mapping points to `/data` in the container.
- **`/health` returns `{"ok":true}` but indexing not ready** → give the server a few seconds after startup, especially after large imports (it may be rebuilding indexes from disk).
- **Permission errors when snapshotting** → ensure the snapshot path is inside the mapped `/data` volume and writable by the container.
- **CORS errors in the browser** → confirm `TYPESENSE_ENABLE_CORS=true` (or configure CORS at your proxy) and that your client is **not** sending the admin key.

---

## 8) Useful links

- Typesense docs: https://typesense.org/docs/
- Docker image: https://hub.docker.com/r/typesense/typesense
- GitHub: https://github.com/typesense/typesense
- Community forum: https://discuss.typesense.org/
