# @actalog/download-docker-volumes

## Description

GitHub Action designed to securely download Docker volumes from a remote server via SSH, compress them with password protection, and either upload the backup as an artifact or save it locally on the runner. This action is ideal for creating secure backups of Docker volumes in CI/CD pipelines, with flexibility for large volumes using self-hosted runners.

## Features

- Downloads Docker volumes from a remote server using `rsync` over SSH.
- Compresses the volumes into a password-protected `.zip` file.
- Supports two upload modes: artifact (default) or local (for self-hosted runners).
- Uploads the backup as an artifact for later retrieval (artifact mode).
- Saves the backup locally on the runner (local mode, useful for large volumes).
- Masks sensitive inputs (e.g., passwords) to ensure security.
- Cleans up SSH known_hosts after execution to avoid key conflicts.

## Requirements

- The runner must have `sshpass` installed for SSH password authentication to work.
- The remote server must have `rsync` installed for the transfer to work.
- For local mode, use a self-hosted runner to access the saved file.

## Inputs

| Name                     | Description                                      | Required | Default                          |
|--------------------------|--------------------------------------------------|----------|--------------------------------- |
| `host`                   | SSH server host                                  | `true`   |                                  |
| `username`               | SSH username                                     | `true`   |                                  |
| `password`               | SSH password                                     | `true`   |                                  |
| `port`                   | SSH port                                         | `false`  | `22`                             |
| `volumes-path`           | Path to the Docker volumes directory on server   | `false`  | `/var/lib/docker/volumes`        |
| `artifact-retention-days`| Number of days to retain the artifact (only applicable in artifact mode) | `false`  | `7`      |
| `backup-password`        | Password to protect the backup file              | `true`   |                                  |
| `upload-mode`            | Upload mode: "artifact" or "local"               | `false`  | `artifact`                       |
| `local-path`             | Local path to save the backup when mode is local | `false`  | `/tmp/docker-volumes-backup.zip` |

## Outputs

| Name          | Description                                                                                      |
|---------------|------------------------------------------------------------------------------------------------- |
| `backup-path` | Path to the backup file on the runner (local filename in artifact mode, full path in local mode) |

## Example Usage

### Artifact Mode (Default)
```yaml
name: Backup Docker Volumes

on:
  schedule:
    - cron: "0 0 * * *" # Runs daily at midnight
  workflow_dispatch:

jobs:
  backup:
    runs-on: ubuntu-latest

    steps:
      - name: Download and Backup Docker Volumes
        uses: actalog/download-docker-volumes@v1
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USERNAME }}
          password: ${{ secrets.SSH_PASSWORD }}
          backup-password: ${{ secrets.BACKUP_PASSWORD }}
```

### Local Mode (Self-Hosted Runner)
```yaml
name: Backup Docker Volumes Locally

on:
  schedule:
    - cron: "0 0 * * *" # Runs daily at midnight
  workflow_dispatch:

jobs:
  backup:
    runs-on: self-hosted  # Use a self-hosted runner

    steps:
      - name: Download and Backup Docker Volumes Locally
        id: backup
        uses: actalog/download-docker-volumes@v1
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USERNAME }}
          password: ${{ secrets.SSH_PASSWORD }}
          backup-password: ${{ secrets.BACKUP_PASSWORD }}
          upload-mode: local
          local-path: /path/to/backup/docker-volumes-backup.zip

      - name: Log Backup Path
        run: echo "Backup saved at ${{ steps.backup.outputs.backup-path }}"
```

## Security

- All sensitive inputs (e.g., `password`, `backup-password`) are masked in logs to prevent exposure.
- Ensure that secrets are stored securely in GitHub Secrets.
