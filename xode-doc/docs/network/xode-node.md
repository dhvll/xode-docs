---
sidebar_position: 4
title: XODE Node
description: Run a XODE node on Ubuntu or using Podman containers
---

# XODE Node

## Running a XODE node directly on the host

### Minimum instance specifications (AWS reference)

- vCPU: 2 (e.g., t2.medium)
- RAM: 4 GB
- Storage: 500 GB
- OS: Ubuntu 22.04 or higher

### Step 1: Download the xode-node binary

#### x86-64
```bash
curl -L "https://drive.usercontent.google.com/download?id=10zStcLL08V3hiCy507CBXMCKCb2VFQsM&confirm=xxx" -o xode-node
```

#### aarch64
```bash
curl -L "https://drive.usercontent.google.com/download?id=1S8uBEuaZhSfJMCwKvbzWXAw_7J4EgPVE&confirm=xxx" -o xode-node
```

### Step 2: Download the XODE chainspec
```bash
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=19C8s1MdVubYjMFiLBmvwhWxTK6bPeyve' -O raw-xode-node-chainspec.json
```

### Step 3: Make the binary executable
```bash
chmod +x xode-node
```

### Step 4: Create a base path directory
```bash
mkdir -p ~/xode
```

### Step 5: Create a startup script

Create `~/xode-node.sh` with the following content:

```bash
#!/bin/bash

/home/ubuntu/xode-node \
  --chain /home/ubuntu/raw-xode-node-chainspec.json \
  --base-path /home/ubuntu/xode \
  --rpc-port 9944 \
  --pruning archive \
  --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
  --name "xode-node-name"
```

Make it executable:
```bash
chmod +x ~/xode-node.sh
```

### Step 6: Test run
```bash
./xode-node.sh
```

### Step 7: Create a systemd service

Create `/etc/systemd/system/xode-collator.service`:

```ini
[Unit]
Description=Xode Node

[Service]
ExecStart=/home/ubuntu/xode-node.sh
Restart=always
RestartSec=120

[Install]
WantedBy=multi-user.target
```

Enable and start:
```bash
sudo systemctl enable xode-collator.service
sudo systemctl start xode-collator.service
```

### Step 8: Verify logs and telemetry
```bash
journalctl -f -u xode-collator | cat
```

Telemetry (XODE chain):

`https://telemetry.polkadot.io/#/0x28cc1df52619f4edd9f0389a7e910a636276075ecc429600f1dd434e281a04e9`

---

## Running a XODE node using a container (Podman)

### Step 1: Install Podman
```bash
sudo apt-get update && sudo apt-get -y install podman
```

### Step 2: Pull the image

Images:
- x86-64: `https://hub.docker.com/r/xoderockson/xode-node-x86_64`
- aarch64: `https://hub.docker.com/r/xoderockson/xode-node-aarch64`

Commands:
```bash
# x86-64
podman pull docker.io/xoderockson/xode-node-x86_64

# aarch64
podman pull docker.io/xoderockson/xode-node-aarch64
```

### Step 3: Prepare a local data directory and run

This directory persists the node database:
```bash
mkdir -p ./xode
```

Run the container (replace NODENAME):
```bash
# x86-64
podman run -d --name xode-node --volume ./xode:/xode -it xoderockson/xode-node-x86_64 <NODENAME>

# aarch64
podman run -d --name xode-node --volume ./xode:/xode -it xoderockson/xode-node-aarch64 <NODENAME>
```

### Step 4: Generate a user service for Podman
```bash
podman generate systemd --new --name xode-node > ./xode-node.service
mkdir -p ~/.config/systemd/user
mv ./xode-node.service ~/.config/systemd/user/
systemctl --user enable xode-node.service
```

### Step 5: Optional — enable auto login (console)

Edit `/lib/systemd/system/getty@.service` and change:

```ini
ExecStart=-/sbin/agetty -o '-p -- \u' --noclear %I $TERM
```

to:

```ini
ExecStart=-/sbin/agetty --noissue --autologin [USERNAME] %I $TERM
```

### Maintaining the container (Podman)

```bash
# Stop all containers
podman stop --all

# Prune container cache
podman container prune -f

# List running containers and images
podman ps
podman images
```


