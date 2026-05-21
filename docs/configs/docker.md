# Docker

## Docker Logging and Rotation
By default, some Docker installations do not configure log rotation. This can lead to uncontrolled log growth and high disk usage over time. This document describes how to configure and verify Docker log rotation to prevent such issues.


### 1. Configure Docker Daemon Logging

Create or update the Docker daemon configuration file:
```bash
/etc/docker/daemon.json
```

Add the following content:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

### 2. Configuration Explanation

| Parameter   | Description |
|-------------|-------------|
| `log-driver` | Sets the logging driver used by Docker. `json-file` stores logs in JSON format on disk. |
| `max-size`   | Maximum size of a single log file before rotation occurs (e.g., `10m` = 10 megabytes). |
| `max-file`   | Maximum number of rotated log files to keep per container. Older logs are removed automatically. |

### 3. Restart Docker Service

After updating the configuration, restart Docker to apply changes:

```bash
systemctl restart docker
```

### 4. Important: Recreate Existing Containers
Log rotation settings only apply to **newly created containers.**

Existing containers will continue using their original logging configuration.

To apply the new settings, you must recreate containers:

```bash
docker stop <container>
docker rm <container>
docker run ...
```

### 5. Verify Configuration

Check whether log rotation is applied:
```bash
docker inspect <container> | grep -A5 LogConfig
```

Expected output:

```json
"LogConfig": {
    "Type": "json-file",
    "Config": {
        "max-size": "10m",
        "max-file": "3"
    }
}
```

### 6. Notes
* If `Config` is empty (`{}`), log rotation is not applied.
* Log rotation does not affect existing large log files; they must be truncated or recreated.
* Consider using the `local` logging driver for improved performance and automatic rotation support.
