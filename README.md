# Plane Docker Compose Setup

This repository contains a Portainer-ready Docker Compose configuration for [Plane](https://plane.so/), an open-source project management tool. It uses the `makeplane/plane-aio-community` all-in-one image and provisions PostgreSQL, Valkey (Redis), RabbitMQ, and MinIO as external stateful services to ensure robustness.

## Features

- **Automated Deployments:** GitHub Actions automatically sync the configuration to a `deploy` branch.
- **Portainer Webhooks:** Deployments are orchestrated by Portainer via Webhooks.
- **Traefik Integration:** Connects seamlessly to a Traefik reverse proxy for HTTPS auto-configuration.
- **Image Vendoring:** A scheduled GitHub Action automatically vendors the `makeplane/plane-aio-community` image to your GitHub Container Registry (`ghcr.io`), adding a layer of stability and resilience against upstream outages.

## Quick Start

1. **Push to GitHub**
   Initialize this repository and push to GitHub:
   ```bash
   git init
   git add .
   git commit -m "init"
   git remote add origin https://github.com/korjavin/plane-compose.git
   git push -u origin master
   ```

2. **Add GitHub Secret**
   In your GitHub repository, go to `Settings` → `Secrets and variables` → `Actions` → `New repository secret`.
   - Name: `PORTAINER_REDEPLOY_HOOK`
   - Value: The webhook URL provided by Portainer (you'll get this in step 3).

3. **Configure Portainer**
   Create a new stack in Portainer using this repository:
   - **Repository URL:** `https://github.com/korjavin/plane-compose.git`
   - **Branch:** `deploy` (Wait for the first GitHub Action run to create this branch, or manually run the deploy workflow).
   - **Compose path:** `docker-compose.yml`
   - **Environment variables:** Copy values from `.env.example` and set strong secrets for `POSTGRES_PASSWORD`, `SECRET_KEY`, `MINIO_ROOT_PASSWORD`, and `RABBITMQ_PASSWORD`.
   - **Webhook:** Enable "Git repository updates webhook" and copy the URL to your GitHub secret.

4. **Trigger Deployment**
   Push to the `master` branch or manually run the GitHub Action to trigger a deployment.

## Environment Variables

| Variable | Description |
|----------|-------------|
| `SERVICE_IMAGE` | Your vendored GHCR image path (e.g. `ghcr.io/korjavin/plane-vendor:latest`) |
| `SERVICE_HOST` | Domain for Traefik routing (e.g., `plane.example.com`) |
| `TRAEFIK_NETWORK_NAME` | The name of your Traefik external network |
| `TRAEFIK_CERTRESOLVER` | The Traefik certificate resolver name |
| `SECRET_KEY` | Plane Django secret key. Generate with `openssl rand -hex 32` |
| `POSTGRES_PASSWORD` | PostgreSQL Database password |
| `RABBITMQ_PASSWORD` | RabbitMQ AMQP password |
| `MINIO_ROOT_PASSWORD` | MinIO Storage root password |

## Updates

- To configure Plane or add environment variables, commit changes to `master` and push.
- The `Vendor Images to GHCR` GitHub Action runs weekly to pull the latest upstream image to GHCR and triggers Portainer to pull and redeploy.
