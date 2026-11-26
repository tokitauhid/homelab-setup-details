# Obsidian LiveSync (Self-Hosted CouchDB)

CouchDB backend enabling end-to-end encrypted, real-time bidirectional synchronization for the **Self-hosted LiveSync** community plugin in Obsidian.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `couchdb:3` |
| **Endpoint** | `http://<SERVER_IP>:5984` |
| **Obsidian Plugin** | `Self-hosted LiveSync` |

---

## Obsidian Client Setup

1. In Obsidian, install the **Self-hosted LiveSync** plugin from Community Plugins.
2. In the plugin settings:
   - **URI**: `http://<SERVER_IP>:5984` (or your reverse proxied HTTPS domain).
   - **User**: `${COUCHDB_USER}`
   - **Password**: `${COUCHDB_PASSWORD}`
   - **Database Name**: `obsidian_vault`
3. Click **Check & Prepare Database** -> **Synchronize**.
