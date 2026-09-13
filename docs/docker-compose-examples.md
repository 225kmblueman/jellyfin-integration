# Docker Compose examples — hardware acceleration

This file shows example docker-compose configurations for running Jellyfin with common hardware acceleration options (Intel VAAPI and NVIDIA). These are examples — adapt device paths, driver installation, and user/group permissions for your host.

## Intel (VAAPI) example

```yaml
version: "3.8"
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    ports:
      - "8096:8096"
      - "8920:8920"
    volumes:
      - ./config:/config
      - ./cache:/cache
      - ./media:/media
    devices:
      - /dev/dri:/dev/dri
    environment:
      - JELLYFIN_PublishedServerUrl=https://jellyfin.example.com
    restart: unless-stopped
```

Notes:
- Ensure the host has VAAPI drivers installed (libva, i965 or iHD drivers depending on CPU).
- Expose /dev/dri to the container and enable hardware acceleration in Jellyfin Admin -> Playback -> Transcoding.

## NVIDIA (NVENC) example

```yaml
version: "3.8"
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
    ports:
      - "8096:8096"
      - "8920:8920"
    volumes:
      - ./config:/config
      - ./cache:/cache
      - ./media:/media
    restart: unless-stopped
```

Notes:
- Install the NVIDIA drivers and the NVIDIA Container Toolkit on the host.
- Confirm the container can access NVIDIA devices (nvidia-smi inside the container should work).
- Enable NVENC/NVDEC in Jellyfin transcoding settings when supported.

## Additional tips
- Prefer direct play/direct stream to reduce CPU load. Configure client apps and server bitrates accordingly.
- Use wired Ethernet for the server where possible for best streaming reliability.
