# go-ubot (Docker Builder)

Repository ini hanya untuk **membuild Docker image** dari repo sumber dan **push ke GHCR** secara manual.

## Apa yang dilakukan
- Checkout repo sumber `lubluniky/ubot`
- Build Docker image dari `Dockerfile` di root repo sumber
- Push image ke `ghcr.io/banghasan/go-ubot`
  - Workflow mem-patch base image Go ke `golang:1.25-alpine` dan set `GOTOOLCHAIN=auto` sebagai fallback

## Cara pakai (manual)
1. Buka tab **Actions** di GitHub repo ini.
2. Jalankan workflow **Build & Push GHCR (manual)**.
3. Isi input:
   - `ref`: branch/tag/commit dari repo sumber (default `main`)
   - `tag`: tag image (default `latest`)

Contoh:
- `ref = main`
- `tag = latest`

## Output
Image akan tersedia di GHCR:
- `ghcr.io/banghasan/go-ubot:<tag>`

## Contoh docker-compose (pakai image GHCR)
File: `docker-compose.yml`

```yaml
version: '3.8'

services:
  ubot:
    image: ghcr.io/banghasan/go-ubot:${UBOT_TAG:-latest}
    container_name: ubot
    restart: unless-stopped

    # Mount config and workspace
    volumes:
      - ~/.ubot:/home/ubot/.ubot

    # Expose gateway port (local only)
    ports:
      - "127.0.0.1:18790:18790"

    # Environment variables (optional overrides)
    environment:
      - TZ=${TZ:-UTC}

    # Security options
    security_opt:
      - no-new-privileges:true

    # Read-only root filesystem (data stored in volume)
    read_only: true
    tmpfs:
      - /tmp:size=64M,mode=1777

    # Resource limits (for compose v2)
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 128M

    # Health check
    healthcheck:
      test: ["CMD", "ubot", "status"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
```

### Cara menjalankan (sesuai repo sumber)
Jalankan gateway (default command):  
`docker compose up -d`

Menjalankan gateway dengan tag tertentu:
`UBOT_TAG=latest docker compose up -d`

Interactive chat:
`docker compose run --rm ubot agent`

### Atasi error permission denied
Jika muncul error seperti `failed to create sessions directory`:

```bash
mkdir -p ~/.ubot/workspace ~/.ubot/sessions ~/.ubot/workspace/memory
sudo chown -R "${USER}":"${USER}" ~/.ubot
```

Compose juga sudah diset `user: "${UID}:${GID}"` agar permission konsisten.

## Contoh konfigurasi (sesuai repo sumber)
File config default di container: `~/.ubot/config.json`  
Karena volume di-mount ke `~/.ubot`, buat file di host:

```json
{
  "agents": {
    "defaults": {
      "model": "anthropic/claude-sonnet-4-20250514",
      "maxTokens": 4096,
      "temperature": 0.7
    }
  },
  "providers": {
    "openrouter": { "apiKey": "sk-or-v1-xxx" },
    "copilot": { "enabled": true, "accessToken": "gho_xxx" },
    "anthropic": { "apiKey": "sk-ant-xxx" },
    "openai": { "apiKey": "sk-xxx" },
    "ollama": { "apiBase": "http://localhost:11434/v1" }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "123456:ABC...",
      "allowFrom": ["your_user_id"]
    }
  },
  "tools": {
    "web": {
      "search": { "apiKey": "BSA..." }
    }
  },
  "mcp": {
    "servers": []
  }
}
```

Catatan:
- Sesuaikan `model` dan provider sesuai kebutuhan.
- Jika hanya pakai satu provider, yang lain bisa dihapus.

## Contoh konfigurasi minimal (OpenAI + Telegram)
```json
{
  "agents": {
    "defaults": {
      "model": "openai/gpt-4o-mini",
      "maxTokens": 2048,
      "temperature": 0.7
    }
  },
  "providers": {
    "openai": { "apiKey": "sk-xxx" }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "123456:ABC...",
      "allowFrom": ["your_user_id"]
    }
  }
}
```

## Catatan
- Repo sumber publik, jadi tidak butuh token khusus untuk checkout.
- Jika nanti repo sumber jadi private, tambahkan `SOURCE_REPO_PAT` pada Secrets dan gunakan di langkah checkout.
