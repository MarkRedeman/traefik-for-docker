# 🛡️ Traefik Reverse Proxy Setup with Automatic HTTPS

This repository contains a minimal and production-ready [Traefik](https://doc.traefik.io/traefik/) configuration using Docker Compose. It automatically issues and renews **Let's Encrypt HTTPS certificates** via the **HTTP-01 challenge** and serves as a reverse proxy for your self-hosted Dockerized services.

## 📦 Folder Structure

```
.
├── docker-compose.yml # Traefik + example app
└── data/acme.json # TLS certificates stored here (auto-created)
```

## 🔧 Key Features

- ✅ **Automatic HTTPS** via Let's Encrypt (HTTP-01 challenge)
- 🔐 **Default TLS** configuration (no need to repeat `.tls=true` or `.certresolver=...`)
- 🔁 **Automatic redirect** from HTTP to HTTPS
- 🐳 **Docker provider**: dynamically routes services based on labels
- 🛡️ Optional secure dashboard via Basic Auth

---

## 🌐 How It Works

1. **Traefik runs as a Docker container**, exposing ports 80 and 443.
2. It watches other Docker containers and automatically configures itself using their labels.
3. When a container is labeled with a domain name (e.g. `Host(\`nginx.example.com\`)`), Traefik:
   - Requests an HTTPS certificate from Let's Encrypt using the HTTP challenge
   - Redirects all HTTP traffic to HTTPS
   - Proxies incoming traffic to the appropriate container

### Example: NGINX Container

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.nginx.rule=Host(`nginx.example.com`)"
  - "traefik.http.routers.nginx.entrypoints=https"
```

That's all you need. No TLS settings required — it's handled globally.

## 🚀 Getting Started

1. Clone or copy this repo
```bash
git clone https://github.com/your-org/your-traefik-setup.git
cd your-traefik-setup
```

2. Create the Docker network
```bash
docker network create traefik-proxy
```

3. Create and secure the certificate storage file
```bash
mkdir -p data
touch data/acme.json
chmod 600 data/acme.json
```

4. Set DNS records
Ensure your domains (e.g., `nginx.example.com`) have `A` records pointing to your VPS IP address.

5. Create a .env file
Create a `.env` file in this directory to store environment variables for the dashboard:
```dotenv
DASHBOARD_HOST=traefik.example.com

# Generate password with `htpasswd -nbB admin yourpassword`
BASIC_AUTH_USER=admin

# Note: double $$ needed to escape $ in Compose YAML
BASIC_AUTH_PASS=$$2y$$05$$YourHashedPasswordHere
```
To generate the hashed password, run:

```bash
htpasswd -nbB admin yourpassword | sed -E 's/^(admin:)(.*)$/\1\2/' | sed 's/\$/\$\$/g'
```
Copy the hash part (after the colon) into `BASIC_AUTH_PASS`.

6. Launch Traefik and your services
```bash
docker compose up -d
```

Visit your domain in the browser — you should be automatically redirected to HTTPS with a valid TLS certificate.

## 📖 Useful Documentation

- [🔗 Traefik Documentation](https://doc.traefik.io/traefik/)
- [🔐 Let’s Encrypt HTTP Challenge](https://letsencrypt.org/docs/challenge-types/#http-01-challenge)
- [📦 Traefik Docker Provider](https://doc.traefik.io/traefik/providers/docker/)
- [🧩 Traefik Routing Configuration](https://doc.traefik.io/traefik/routing/routers/)
- [🛡️ Basic Auth Middleware](https://doc.traefik.io/traefik/middlewares/http/basicauth/)

## 🔐 Optional: Securing the Dashboard

The Traefik dashboard is available at the host defined by the `DASHBOARD_HOST` environment variable (e.g., `https://traefik.example.com``) and protected by Basic Auth credentials defined in `.env`.

Make sure your DNS points `traefik.example.com` (or your chosen host) to your server IP.

## 🧼 Maintenance Tips
- Certificates are automatically renewed before expiration.
- Restart Traefik if you ever rotate the `acme.json` file or make major config changes.
- Run `docker logs traefik` to inspect the proxy and certificate behavior.

## 👥 Contributing
Feel free to fork and modify this setup for your team or environment. Issues and pull requests welcome!

## 📝 License
MIT – do as you please.
