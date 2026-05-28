# Tycho Backup Receiver Service

A lightweight, secure containerized SSH and `rsync` server designed to act as a remote target for Tycho backups.

## Configuration & Variables

Configure these variables in your target environment's `.env` file before installing:

| Variable | Description | Default |
|---|---|---|
| `BACKUP_RECEIVER_PORT` | Host SSH port to expose for backup client connections. | `2222` |
| `BACKUP_RECEIVER_PUBLIC_KEY` | SSH public key of the client (the server running Tycho to back up). | *(Required)* |

## Client SSH Key Generation

On the client machine (the one being backed up), if you don't have an SSH key pair already, generate one:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_tycho -N ""
```

Extract the public key content:

```bash
cat ~/.ssh/id_rsa_tycho.pub
```

Copy the output and paste it as the value of `BACKUP_RECEIVER_PUBLIC_KEY` on the receiver host.

## Client Integration Examples

Once the receiver is deployed, you can run a remote cron backup from the client machine:

```bash
# Perform a full metadata backup to the receiver
tycho backup \
  --remote backup@<receiver_ip>:/backups \
  --port 2222 \
  --identity ~/.ssh/id_rsa_tycho \
  --keep 5 \
  --non-interactive
```

For ultra-fast, remote incremental backups using `rsync`:

```bash
tycho backup \
  --remote backup@<receiver_ip>:/backups \
  --incremental \
  --port 2222 \
  --identity ~/.ssh/id_rsa_tycho \
  --non-interactive
```
