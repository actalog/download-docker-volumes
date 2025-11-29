# @actalog/download-docker-volumes

## Description

GitHub Action designed to securely download Docker volumes from a remote server via SSH, compress them with password protection, and upload the backup as an artifact. This action is ideal for creating secure backups of Docker volumes in CI/CD pipelines.

## Features

- Downloads Docker volumes from a remote server using `scp`.
- Compresses the volumes into a password-protected `.zip` file.
- Uploads the backup as an artifact for later retrieval.
- Masks sensitive inputs (e.g., passwords) to ensure security.

## Inputs

| Name                     | Description                                      | Required | Default                     |
|--------------------------|--------------------------------------------------|----------|-----------------------------|
| `host`                   | SSH server host                                  | `true`   |                             |
| `username`               | SSH username                                     | `true`   |                             |
| `password`               | SSH password                                     | `true`   |                             |
| `port`                   | SSH port                                         | `false`  | `22`                        |
| `volumes-path`           | Path to the Docker volumes directory on server   | `false`  | `/var/lib/docker/volumes`   |
| `artifact-retention-days`| Number of days to retain the artifact            | `false`  | `7`                         |
| `backup-password`        | Password to protect the backup file              | `true`   |                             |

## Example Usage

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

## Security

- All sensitive inputs (e.g., `password`, `backup-password`) are masked in logs to prevent exposure.
- Ensure that secrets are stored securely in GitHub Secrets.
