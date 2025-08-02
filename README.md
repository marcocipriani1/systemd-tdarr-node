# Tdarr Node Systemd Service

This systemd service file runs a Tdarr Node container using Podman with GPU acceleration support for media transcoding.

## What This Does

Tdarr Node is a distributed transcoding system that processes media files according to rules defined on a Tdarr Server. This service:

- Runs a Tdarr Node container that connects to a central Tdarr Server
- Provides GPU acceleration using NVIDIA hardware
- Automatically restarts on failure
- Manages container lifecycle through systemd
- Handles media transcoding tasks distributed from the server

## Prerequisites

- Podman installed and configured
- NVIDIA GPU with drivers installed
- NVIDIA Container Toolkit configured for Podman
- Access to a running Tdarr Server instance
- Proper directory structure created

## Directory Structure

Before deploying, ensure these directories exist:

```bash
sudo mkdir -p /srv/tdarr/configs
sudo mkdir -p /srv/tdarr/logs
sudo mkdir -p /media
sudo mkdir -p /tmp/transcode_cache
```

Set appropriate permissions:
```bash
sudo chown -R 1000:1000 /srv/tdarr/
sudo chown -R 1000:1000 /tmp/transcode_cache
```

## Configuration

### Required Changes

Edit the service file and modify these key variables:

#### Network Configuration
- `serverIP={server-ip}` - IP address of your Tdarr Server
- `serverPort=8266` - Port of your Tdarr Server (default: 8266)

#### Node Identity
- `nodeName={your-nodename}` - Unique name for this node
- `apiKey=""` - API key from your Tdarr Server (if authentication enabled)

#### Resource Allocation
- `transcodegpuWorkers=3` - Number of GPU transcoding workers
- `transcodecpuWorkers=0` - Number of CPU transcoding workers
- `healthcheckgpuWorkers=0` - GPU workers for health checks
- `healthcheckcpuWorkers=2` - CPU workers for health checks

#### User Configuration
- `PUID=1000` - User ID for file permissions
- `PGID=1000` - Group ID for file permissions
- `TZ=Europe/Rome` - Timezone setting

### Volume Mounts

Configure these paths according to your setup:

- `/srv/tdarr/configs:/app/configs:rw` - Tdarr configuration files
- `/srv/tdarr/logs:/app/logs:rw` - Log files
- `/media:/media:rw` - Media files location
- `/tmp/transcode_cache:/transcode_cache:rw` - Temporary transcoding cache

### Optional Settings

- `priority=-1` - Node priority (-1 to 10, lower = higher priority)
- `pollInterval=2000` - How often to check for new jobs (milliseconds)
- `startPaused=false` - Whether to start the node in paused state
- `maxLogSizeMB=10` - Maximum log file size in MB
- `ffmpegVersion=7` - FFmpeg version to use

## Installation

1. **Save the service file**
   ```bash
   sudo cp tdarr-node.service /etc/systemd/system/
   ```

2. **Reload systemd daemon**
   ```bash
   sudo systemctl daemon-reload
   ```

3. **Enable the service**
   ```bash
   sudo systemctl enable tdarr-node.service
   ```

4. **Start the service**
   ```bash
   sudo systemctl start tdarr-node.service
   ```

## Management Commands

### Check service status
```bash
sudo systemctl status tdarr-node.service
```

### View logs
```bash
sudo journalctl -u tdarr-node.service -f
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

## GPU Requirements

This configuration requires:
- NVIDIA GPU with proper drivers
- NVIDIA Container Toolkit installed
- Podman configured for GPU access

### Verify GPU Access
```bash
podman run --rm --runtime=nvidia --env NVIDIA_VISIBLE_DEVICES=all nvidia/cuda:11.0-base nvidia-smi
```

## Troubleshooting

### Common Issues

1. **Container fails to start**
   - Check if directories exist and have proper permissions
   - Verify NVIDIA drivers and container toolkit installation
   - Ensure Tdarr Server is accessible

2. **GPU not detected**
   - Verify NVIDIA runtime configuration
   - Check device permissions
   - Ensure NVIDIA Container Toolkit is properly configured

3. **Permission errors**
   - Verify PUID/PGID settings match your user
   - Check directory ownership and permissions

4. **Cannot connect to server**
   - Verify serverIP and serverPort settings
   - Check network connectivity
   - Ensure firewall allows communication

### Log Analysis
```bash
# View service logs
sudo journalctl -u tdarr-node.service --since "1 hour ago"

# View container logs
sudo podman logs tdarr-node
```

## Security Considerations

- Consider using API key authentication for server communication

## Performance Tuning

- Adjust worker counts based on your hardware capabilities
- Monitor GPU/CPU usage and adjust accordingly
- Use fast storage for transcode cache
- Consider network bandwidth when setting worker counts