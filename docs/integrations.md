# Jellyfin Integration & Casting

This document provides setup and integration guidance for using Jellyfin as a media server to cast or stream content to phones, tablets, smart TVs, and other devices. It focuses on practical steps (including Docker examples), recommended settings, and common troubleshooting tips.

> Official Jellyfin: https://jellyfin.org

## Quick overview
- Jellyfin is a free, open-source media system for streaming your media to clients across your network and the internet.
- Common use cases: run a server at home (or in the cloud), access your library from mobile apps, cast to Chromecast/Smart TV, integrate with TV apps and remotes.

## Table of contents
- Prerequisites
- Installation (Docker recommended)
- Basic server configuration
- Remote access and security (reverse proxy + HTTPS)
- Casting and device integrations
- Transcoding and performance tips
- Mobile apps and TV apps
- Common troubleshooting
- Helpful links

## Prerequisites
- A machine to host Jellyfin (NAS, Linux server, Raspberry Pi 4+, or a cloud VM). For best performance when transcoding, use a machine with hardware acceleration support (Intel QuickSync, NVIDIA NVENC, or AMD VCE).
- Local network with router access for port forwarding (optional for remote access).
- Basic familiarity with Docker or package managers for your OS.

## Installation (Docker - recommended)
Docker keeps Jellyfin isolated and portable. Example docker-compose.yml (minimal):

```yaml
version: "3.8"
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    network_mode: "bridge"
    ports:
      - "8096:8096" # HTTP (change or disable if using reverse proxy)
      - "8920:8920" # HTTPS (optional)
    volumes:
      - ./config:/config
      - ./cache:/cache
      - ./media:/media
    devices:
      # Example for hardware acceleration on Linux (adjust per host)
      - /dev/dri:/dev/dri # Intel VAAPI
    restart: unless-stopped
```

Tips:
- Map media folders into `/media` and point Jellyfin libraries to those paths.
- Use named volumes or host directories for config and cache to persist data.
- For hardware encoding/decoding, ensure host drivers are installed and the device path is exposed to the container.

## Basic server configuration
1. Start the server and open http://<server-ip>:8096 to run the first-time setup wizard. Create an admin account. 
2. Add libraries: point Jellyfin to your media folders (Movies, TV Shows, Music, Photos).
3. Metadata: enable preferred metadata providers and refresh libraries after adding new content.
4. Users: create non-admin accounts for family members and configure access restrictions and parental controls as needed.

## Remote access and security
For safe remote access, run Jellyfin behind a reverse proxy (Nginx, Caddy, Traefik) with HTTPS.

Example Caddyfile (automatic HTTPS):

```
jellyfin.example.com {
  reverse_proxy 127.0.0.1:8096
  header_up Host {host}
  header_up X-Real-IP {remote}
  header_up X-Forwarded-For {remote}
  header_up X-Forwarded-Proto {scheme}
}
```

Security notes:
- Use strong admin passwords and create separate user accounts for daily use.
- Enable HTTPS to protect credentials and streams. If using a cloud VM, keep the management ports closed and use SSH for admin tasks.
- Consider IP allowlists or VPNs for more restricted access.

## Casting and device integrations
Jellyfin supports many client apps and casting mechanisms:

- Jellyfin clients: Official apps exist for Android, iOS, Roku (community), Apple TV (community), and many smart TVs via third-party apps. Check supported clients at https://jellyfin.org/clients/.
- Chromecast: Use the Jellyfin Android/iOS app to cast to Chromecast devices. Casting uses direct stream URLs from the server and supports transcoding if needed.
- DLNA/UPnP: Jellyfin has a DLNA server mode—enable it in Admin -> Networking -> DLNA to allow TVs and game consoles to discover streams.
- AirPlay: Some clients and network setups allow AirPlay mirroring. Native AirPlay streaming from Jellyfin is limited; use client apps or casting as primary methods.
- Smart TV apps & native players: For better UX, install official or community Jellyfin apps on TVs (LG webOS, Samsung Tizen, Android TV) where available.

Casting tips:
- Use wired Ethernet for the server when possible to reduce buffering and improve transcoding reliability.
- For Chromecast, enable direct play/direct stream where possible to avoid unnecessary transcoding.
- If casting to a TV with native support, prefer the TV’s app over casting for better stability.

## Transcoding and performance tips
- Direct play is ideal: ensure your clients can play the media container and codecs used to avoid server-side transcoding.
- Hardware acceleration: enable in Admin -> Playback -> Transcoding and configure the appropriate device. For Docker, expose device paths (/dev/dri or NVIDIA devices) and install the host drivers.
- Bitrate & streaming quality: configure per-user streaming limits to prevent saturating the uplink or local network.
- Monitor CPU & RAM: transcoding can be CPU-intensive; use a dedicated transcoding-capable machine for heavier usage.

## Mobile apps and TV apps
- Android: Jellyfin for Android (Cast support, downloads for offline viewing).
- iOS/iPadOS: Official Jellyfin client with streaming and downloads.
- Android TV / Fire TV: Jellyfin Android TV builds available; sideload if needed.
- Third-party clients: Emby and Plex forks/clients may interoperate in some setups—stick with official Jellyfin clients when possible.

## Common troubleshooting
- Playback stutters/buffers: Check server CPU, transcoding activity, network bandwidth, and client capabilities. Try direct play and lower bitrate settings.
- Library scanning issues: Confirm file system permissions for the Jellyfin process/container and check logs for scanner errors.
- Remote access failing: Verify reverse proxy settings, firewall rules, and that port forwarding (if used) points to the correct host.
- Subtitles not showing: Ensure the client supports the subtitle format or enable burning subtitles via transcoding.

## Useful commands & logs
- Docker logs: docker logs -f jellyfin
- Systemd: journalctl -u jellyfin -f (if running as a service)
- Server logs: check the /config/log/ directory inside the container or host config path for detailed errors.

## Helpful links
- Jellyfin official site: https://jellyfin.org
- Jellyfin documentation: https://jellyfin.org/docs/
- Jellyfin clients: https://jellyfin.org/clients/
- Community support: https://forum.jellyfin.org/

If you want, I can:
- Create a branch and open a pull request for this file instead of committing to main.
- Add a short sample docker-compose with hardware acceleration notes for Intel/NVIDIA.
- Add a section on integrating Jellyfin with your app (OAUTH, embedding player links, or generating tokenized URLs).

— GitHub Copilot Chat Assistant
