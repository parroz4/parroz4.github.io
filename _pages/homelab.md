---
title: "Raspberry Pi 5 Homelab"
permalink: /projects/homelab/
toc: true
toc_sticky: true
---

A production-grade home server running **24 containerized services** on a single Raspberry Pi 5 (8 GB), fully managed with Docker Compose and secured through Cloudflare Zero Trust.

[View on GitHub](https://github.com/parroz4/homelab-pi5){: .btn .btn--inverse}

## Hardware

| Component | Specification |
|-----------|---------------|
| SBC | Raspberry Pi 5, 8 GB RAM |
| OS Storage | 64 GB SD Card |
| Data Storage | 1 TB External SSD (USB 3.0) |
| Network | Gigabit Ethernet |
| OS | Raspberry Pi OS (Debian 12 Bookworm, 64-bit) |

## Architecture

```
INTERNET
   |
   v
Cloudflare (WAF + Tunnel)
   |
   v
Raspberry Pi 5
   ├── Pi-hole DNS
   ├── Cloudflared Tunnel
   └── Docker Containers (24 services)
          |
          v
   External SSD (Data Store)
```

All traffic enters through a Cloudflare Tunnel -- no ports are exposed on the router. Pi-hole provides network-wide DNS filtering, and Docker containers are isolated with dedicated networks.

## Services

### Media and Storage (6)

| Service | Purpose | Why Chosen |
|---------|---------|------------|
| Immich | Photo management | Google Photos alternative with full AI features |
| Jellyfin | Media streaming | Open-source Plex alternative |
| Paperless-ngx | Document management | OCR and full-text search |
| Filebrowser | Web file manager | Simple, lightweight |
| Syncthing | P2P file sync | No cloud dependency |
| Sync-in | File sync server | Alternative sync with MariaDB backend |

### Monitoring and Observability (8)

| Service | Purpose | Why Chosen |
|---------|---------|------------|
| Beszel | System metrics | Lightweight, Pi-optimized |
| Uptime Kuma | Service monitoring | Beautiful UI, flexible alerts |
| NetAlertX | Network discovery | Track all devices |
| WUD | Container updates | Know when to update |
| LoggiFly | Log monitoring | Real-time Telegram alerts |
| Diun | Docker image updates | Notifies on new versions |
| Speedtest Tracker | ISP monitoring | Historical bandwidth data |
| ChangeDetection | Website monitoring | Track external changes |

### Networking and Security (2)

| Service | Purpose | Why Chosen |
|---------|---------|------------|
| Pi-hole | DNS + ad-blocking | Network-wide protection |
| Cloudflared | Secure tunnel | Zero exposed ports |

### Management (3)

| Service | Purpose | Why Chosen |
|---------|---------|------------|
| Portainer | Container UI | Visual management |
| Homepage | Dashboard | Single pane of glass |
| Backrest | Backup orchestration | Restic made easy |

### Automation (2)

| Service | Purpose | Why Chosen |
|---------|---------|------------|
| Home Assistant | Smart home | Local-first automation |
| N8N | Workflow automation | Self-hosted Zapier |

### Utilities (3)

| Service | Purpose |
|---------|---------|
| Warracker | Warranty tracking |
| iSponsorBlockTV | SponsorBlock for TVs |
| Kaneo | Project management |

## Architecture Decisions

### Cloudflare Tunnel vs Traditional Reverse Proxy

I chose Cloudflare Tunnel instead of exposing ports with nginx or Traefik. This means zero open ports on the router, free SSL certificates and WAF, built-in DDoS protection, and geo-restriction capabilities. The trade-off is a dependency on Cloudflare, but the security benefits outweigh that for home use.

### External SSD for Data, SD Card for OS

SD cards have limited write cycles, so all database and media data lives on the external SSD. This provides better I/O performance and makes it easy to backup or migrate data independently from the OS.

### Multi-layer Monitoring

Rather than a single all-in-one monitoring solution, I run multiple specialized tools: Beszel for low-overhead system metrics, Uptime Kuma for service availability, LoggiFly for real-time log analysis, and WUD for container update tracking. Each tool excels at its specific job.

### Telegram for Alerting

All notifications go through a Telegram bot. It is free, reliable, delivers instant push notifications, works globally without self-hosting, and has an easy API integration with support for rich formatting.

## Security

The homelab uses six security layers:

1. **Cloudflare WAF + DDoS Protection** -- all traffic filtered before reaching the Pi
2. **Cloudflare Access (OAuth)** -- authentication at the edge
3. **Geo-restriction** -- Italy only for most services
4. **Pi-hole DNS filtering** -- blocks ads and malicious domains network-wide
5. **Container isolation** -- dedicated Docker networks per service group
6. **Secrets management** -- no default passwords, all secrets in `.env` files (gitignored), automated sanitization before commits

## Backup Strategy

Following the 3-2-1 rule, managed through Backrest with Restic:

| Data Type | Local Copy | Cloud Copy | Frequency |
|-----------|-----------|------------|-----------|
| Docker configs | SSD | Backblaze B2 | Daily |
| Databases | SSD | Backblaze B2 | Daily |
| Photos (Immich) | SSD | Backblaze B2 | Weekly |
| Documents | SSD | Backblaze B2 | Daily |

All backups are incremental, encrypted, and deduplicated.

## Skills Demonstrated

| Area | Technologies |
|------|-------------|
| Containerization | Docker, Docker Compose, container networking |
| Networking | DNS (Pi-hole), Cloudflare Tunnel, OAuth |
| Monitoring | Beszel, Uptime Kuma, LoggiFly, WUD, Telegram alerts |
| Automation | Home Assistant, N8N workflows, bash scripting |
| Security | Secret management, geo-restriction, HTTPS everywhere |
| Backup | Restic, Backrest, Backblaze B2, 3-2-1 strategy |
| Linux Admin | Debian/Raspberry Pi OS, systemd, permissions, storage |

## What is Next

- Kubernetes migration (k3s) for learning container orchestration
- Prometheus + Grafana stack for deeper metrics
- Automated testing for compose files
- Terraform for Cloudflare configuration
