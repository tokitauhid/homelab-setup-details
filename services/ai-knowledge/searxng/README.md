# SearXNG (Privacy Metasearch Engine)

SearXNG is a privacy-respecting, open-source metasearch engine that aggregates results from over 70 search services while ensuring users are not tracked or profiled.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `searxng/searxng:latest` |
| **Backend Cache** | `redis:alpine` |
| **Web Interface** | `http://<SERVER_IP>:8081` |

---

## Configuration & Deployment

1. Prepare configuration directory:
   ```bash
   mkdir -p etc-searxng redis-data
   cp settings.yml.example etc-searxng/settings.yml
   ```
2. Generate a secure secret key and set `.env`:
   ```bash
   cp .env.example .env
   sed -i "s/change_this_to_a_random_32_byte_hex_key/$(openssl rand -hex 32)/g" .env
   ```
3. Start the containers:
   ```bash
   docker compose up -d
   ```
4. Access `http://<SERVER_IP>:8081` to perform searches across Google, Bing, DuckDuckGo, Reddit, Wikipedia, and more.
