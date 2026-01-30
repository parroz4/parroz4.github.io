---
title: "Building My Raspberry Pi 5 Homelab: 24 Services on a Single Board"
date: 2026-01-29
categories:
  - homelab
tags:
  - raspberry-pi
  - docker
  - self-hosted
  - homelab
excerpt: "How I turned a Raspberry Pi 5 into a production-grade home server running 24 containerized services with Docker Compose and Cloudflare Zero Trust."
---

I recently finished building out my homelab on a Raspberry Pi 5 (8 GB). It now runs 24 Docker containers covering media management, monitoring, networking, automation, and more. Here is how it came together and what I learned along the way.

## Why a Raspberry Pi?

The 8 GB model hits a sweet spot: enough RAM for serious workloads, low power consumption (under 10W), completely silent, and cheap enough that failures are not catastrophic. The constraints of a single-board computer also force good architectural decisions -- you cannot brute-force your way past limited resources, so every service has to earn its place.

## The Stack

All 24 services run as Docker containers managed through Docker Compose. Here is the breakdown by category:

**Media and Storage (6 services):** Immich for photos, Jellyfin for media streaming, Paperless-ngx for documents, Filebrowser, Syncthing, and Sync-in for file management and synchronization.

**Monitoring (8 services):** This is where I went deepest. Beszel tracks system metrics with minimal overhead. Uptime Kuma monitors service availability. LoggiFly watches container logs in real-time and sends Telegram alerts. Diun and WUD track Docker image updates. NetAlertX discovers new network devices. Speedtest Tracker keeps historical ISP performance data. ChangeDetection watches external websites for changes.

**Networking (2 services):** Pi-hole for DNS-based ad-blocking across the whole network, and Cloudflared to create a secure tunnel to Cloudflare -- no ports exposed on my router.

**Management (3 services):** Portainer for container management, Homepage as a dashboard, and Backrest for backup orchestration using Restic.

**Automation (2 services):** Home Assistant for smart home control and N8N for workflow automation.

**Utilities (3 services):** Warracker for warranty tracking, iSponsorBlockTV for blocking sponsors on smart TVs, and Kaneo for project management.

## Key Decisions

The most impactful decision was choosing **Cloudflare Tunnel** over a traditional reverse proxy like nginx or Traefik. With a tunnel, zero ports are open on my router. Cloudflare handles SSL, WAF, DDoS protection, and geo-restriction for free. The trade-off is a dependency on a third-party service, but for a home setup the security benefits are substantial.

I also split storage across two tiers: the **SD card handles the OS** while a **1 TB external SSD** stores all data (databases, media, configs). SD cards have limited write cycles, so keeping heavy I/O on the SSD extends the life of the boot drive and improves performance.

For monitoring, I deliberately chose **multiple specialized tools** instead of a single all-in-one solution. Each tool does one thing well, and together they cover system metrics, service uptime, log analysis, and container updates. Every alert goes to **Telegram**, which is free, reliable, and delivers instant push notifications.

## Lessons Learned

**RAM is the real bottleneck.** On an 8 GB Pi, you have to be selective about what runs. I learned to check each service's memory footprint before deploying and to set resource limits in Compose files.

**Monitor your monitoring.** LoggiFly catches issues in other services before they escalate. Having alerts on alerts sounds redundant, but it has saved me more than once.

**Secrets management from day one.** I made the mistake of committing secrets to git early on. Now all secrets live in `.env` files that are gitignored, and a sanitization script runs before every commit to catch any leaks. Git history was cleaned with `git-filter-repo`.

**Start simple, add complexity.** Each of the 24 services was added one at a time. I got each one working reliably before moving on. This incremental approach made debugging much easier than deploying everything at once.

## What is Next

I am planning to explore **k3s** (lightweight Kubernetes) to learn container orchestration beyond Compose. I also want to add **Prometheus + Grafana** for deeper metrics visualization, automated testing for compose files, and **Terraform** for managing Cloudflare configuration as code.

---

For the full service list, architecture details, and configuration, check out the [project page](/projects/homelab/) or the [GitHub repository](https://github.com/parroz4/homelab-pi5).
