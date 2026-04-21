# Fedora CoreOS Home Infrastructure (IaC)

Declarative home server configuration using Fedora CoreOS and Podman Quadlets.

## Core Architecture

This infrastructure uses Butane to generate Ignition files for automated deployment.

- **OS**: [Fedora CoreOS (FCOS)](https://docs.fedoraproject.org/en-US/fedora-coreos/) provides an immutable, atomically updating host.
- **Filesystem**: bcachefs support is provided by [fedora-coreos-bcachefs](https://github.com/piewhat/fedora-coreos-bcachefs). To use it, the system must be rebased after installation.
- **Orchestration**: [Podman Quadlets](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) manage containers as native systemd units. Configuration files are located in `files/containers/`.
- **Provisioning**: [Butane](https://coreos.github.io/butane/) handles the configuration of users, storage, and services.
- **Updates**: See README in [fedora-coreos-bcachefs](https://github.com/piewhat/fedora-coreos-bcachefs), while `podman-auto-update` handles container images.

## Deployed Services

The configuration defined in `butane/server.bu` includes the following stacks.

### Infrastructure & Core

- **Networking**: Caddy (Reverse Proxy), Gluetun (VPN).
- **Databases**: PostgreSQL, Redis.
- **Storage**: MinIO (S3 compatible), Samba & WSDD (File sharing).

### Media & Entertainment

- **Streaming**: Plex, Jellyfin.
- **Music**: Navidrome.
- **Download clients**: qBittorrent, Transmission (with OpenVPN).
- **Management**: Prowlarr, Radarr (HD & 4K), Sonarr (HD & 4K), Lidarr, Jackett, FlareSolverr, Seerr.
- **Automation**: Cleanuparr, Recyclarr.

### Smart Home & Tools

- **Home Automation**: Home Assistant, Mosquitto (MQTT).
- **Security**: Frigate (NVR).
- **Productivity**: Nextcloud, Ente (Photos), Synapse (Matrix).
- **Communication**: Livekit, LK-JWT.
- **Gaming**: Minecraft (mc, hmc), Lavalink.

## Management

Containers run as the rootless `containers` user.

### Build Configuration

```bash
butane --pretty --strict butane/server.bu > server.ign
```

### Managing Services

To interact with user-level services, use `machinectl` to enter the user context.

```bash
# Enter the container user environment
sudo machinectl shell containers@

# Common commands within the shell
systemctl --user status caddy.service
systemctl --user restart plex.service
journalctl --user -u home-assistant.service -f
```

## Storage

Persistent data is stored in `/var/swarm`. SELinux labels are managed via `file_contexts.subs` to ensure compatibility with standard paths while maintaining isolation.
