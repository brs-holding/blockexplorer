# BLOCK Blockchain Explorer

Block explorer for the BLOCK L1 blockchain, powered by [Blockscout](https://www.blockscout.com/).

## 🌐 Live Explorer

**URL:** [http://64.111.92.74](http://64.111.92.74)

## ⛓️ Chain Details

| Parameter | Value |
|-----------|-------|
| Chain Name | BLOCK |
| Chain ID | 9999 |
| Consensus | PoA (Clique) |
| Block Time | 3 seconds |
| Currency | BLOCK |
| RPC URL | http://64.52.80.204:8545 |

## 🏗️ Architecture

```
Users → http://64.111.92.74 → Nginx (port 80)
                                  ├── / → Frontend (port 3000)
                                  ├── /api → Backend (port 4000)
                                  └── /socket → WebSocket (port 4000)
                                        ↓
                                  RPC Node (64.52.80.204:8545)
                                        ↓
                                  Validators (Server 1 + 2)
```

## 🐳 Stack

- **Frontend:** `ghcr.io/blockscout/frontend:latest` (Next.js)
- **Backend:** `blockscout/blockscout:latest` (Elixir)
- **Database:** PostgreSQL 16
- **Cache:** Redis 7
- **Proxy:** Nginx

## 🚀 Deployment

### Prerequisites
- Ubuntu 24.04 server
- Docker + Docker Compose
- Nginx

### Quick Start

1. Clone this repo:
```bash
git clone https://github.com/brs-holding/blockexplorer.git
cd blockexplorer
```

2. Copy files to server:
```bash
scp docker-compose.yml root@YOUR_SERVER:/opt/blockscout/
scp nginx-blockscout.conf root@YOUR_SERVER:/etc/nginx/sites-available/blockscout
```

3. Start containers:
```bash
cd /opt/blockscout
docker compose up -d
```

4. Enable Nginx:
```bash
ln -sf /etc/nginx/sites-available/blockscout /etc/nginx/sites-enabled/blockscout
rm -f /etc/nginx/sites-enabled/default
nginx -t && systemctl restart nginx
```

## 📁 Files

| File | Description |
|------|-------------|
| `docker-compose.yml` | Docker Compose config for all Blockscout services |
| `nginx-blockscout.conf` | Nginx reverse proxy configuration |

## 🔗 Community

- 📱 [Telegram](https://t.me/bl0ck_official)
- 🐦 [Twitter / X](https://x.com/bl0ck_gg)
- 🎵 [TikTok](https://www.tiktok.com/@bl0ck_official)

## 📄 License

MIT
