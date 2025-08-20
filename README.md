# Tdarr Node Systemd Service

This systemd service file runs a Tdarr Node container using Podman with GPU acceleration support for media transcoding.

## What This Does

Tdarr Node is a distributed transcoding system that processes media files according to rules defined on a Tdarr Server,. This service:

- Runs a Tdarr Node container that connects to a central Tdarr Server
- Provides GPU acceleration using NVIDIA hardware
- Automatically restarts on failure
- Manages container lifecycle through systemd
- Handles media transcoding tasks distributed from the server

[Note: Tdarr is developed by HaveAGitGat](https://github.com/HaveAGitGat) This project only provides a Podman systemd unit for running Tdarr Node.

## Why Use This Podman Systemd unit?

- Designed for systems where Docker causes networking issues with VM bridges
- Suitable for setups without a dedicated GPU installed on the media server

## Prerequisites

- Podman installed and configured
- NVIDIA GPU with drivers installed
- [NVIDIA Container Toolkit configured](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) for [Podman](https://podman-desktop.io/docs/podman/gpu)
- Access to a running Tdarr Server instance
- Proper directory structure created

## Directory Structure

Before deploying, ensure these directories exist or choose your own:

```bash
sudo mkdir -p /srv/tdarr/configs
sudo mkdir -p /srv/tdarr/logs
sudo mkdir -p /media
sudo mkdir -p /srv/tdarr/transcode_cache
```

Set appropriate permissions:
```bash
sudo chown -R 1000:1000 /srv/tdarr/
sudo chown -R 1000:1000 /srv/tdarr/transcode_cache
```

## Configuration

#### Network Configuration
- `serverIP={server-ip}` - IP address of your Tdarr Server
- `serverPort=8266` - Port of your Tdarr Server (default: 8266)

#### Node Identity
- `nodeName={your-nodename}` - Unique name for this node

#### Resource Allocation
- `transcodegpuWorkers=3` - Number of GPU transcoding workers (Max 3 if you don't use a professional grade gpu or the Nvidia patch)
- `transcodecpuWorkers=0` - Number of CPU transcoding workers
- `healthcheckgpuWorkers=0` - GPU workers for health checks
- `healthcheckcpuWorkers=4` - CPU workers for health checks

#### User Configuration
- `PUID=1000` - User ID for file permissions
- `PGID=1000` - Group ID for file permissions
- `TZ=Europe/Rome` - Timezone setting

#### Config File
Edit `/srv/tdarr/configs/Tdarr_Node_Config.json`:
- Add folder mappings
- Copy remaining settings from the systemd unit

### Volume Mounts

You can reconfigure these paths according to your setup:

- `/srv/tdarr/configs:/app/configs:rw` - Tdarr configuration files
- `/srv/tdarr/logs:/app/logs:rw` - Log files
- `/media:/media:rw` - Media files location
- `/srv/tdarr/transcode-cache:/temp:rw` - Temporary transcoding cache

### Optional Settings

- `ExecStartPre=/bin/sleep 60` - Delay the start of the Tdarr Node service by 60 seconds to ensure that all network mounts, remove if you use a local dir
- `priority=-1` - Node priority (-1 to 10, lower = higher priority)
- `pollInterval=2000` - How often to check for new jobs (milliseconds)
- `startPaused=false` - Whether to start the node in paused state
- `maxLogSizeMB=10` - Maximum log file size in MB
- `ffmpegVersion=7` - FFmpeg version to use

## Installation

1. **Copy the service file**
```bash
sudo cp tdarr-node.service /etc/systemd/system/
```

2. **Reload systemd daemon**
```bash
sudo systemctl daemon-reload
```

3. **Enable the service**
```bash
sudo systemctl enable --now tdarr-node.service
```

## Management Commands

### Check service status
```bash
sudo systemctl status tdarr-node.service
```

### View logs
```bash
sudo journalctl -u tdarr-node -b --no-pager
```

### Stop the service
```bash
sudo systemctl stop tdarr-node.service
```

### Restart the service
```bash
sudo systemctl restart tdarr-node.service
```

### Disable the service
```bash
sudo systemctl disable tdarr-node.service
```

## Uninstall

1. **Stop the service**
```bash
sudo systemctl stop tdarr-node.service
```

2. **Disable the service**
```bash
sudo systemctl disable tdarr-node.service
```

3. **Delete the service file**
```bash
sudo rm /etc/systemd/system/tdarr-node.service
```

4. **Delete tdarr-node container**
```bash
sudo podman rm tdarr-node
```

## Security Considerations

- Consider using API key authentication for server communication