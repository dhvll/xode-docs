---
sidebar_position: 5
title: XODE Node (Kusama 3344)
description: Run an XODE node for Kusama 3344 with systemd and RPC options
---

# XODE Node (Kusama 3344)

## Requirements

- 8 Cores (Core i9 or equivalent)
- 32 GB RAM
- 1 TB SSD
- Ubuntu 24.04 LTS

---

## Step 1: Download the xode-node binary

### x86-64
```bash
curl -L "https://drive.usercontent.google.com/download?id=10zStcLL08V3hiCy507CBXMCKCb2VFQsM&confirm=xxx" -o xode-node
```

### aarch64
```bash
curl -L "https://drive.usercontent.google.com/download?id=1S8uBEuaZhSfJMCwKvbzWXAw_7J4EgPVE&confirm=xxx" -o xode-node
```

## Step 2: Download the XODE Kusama 3344 chainspec
```bash
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=19C8s1MdVubYjMFiLBmvwhWxTK6bPeyve' -O raw-xode-node-chainspec.json
```

## Step 3: Make the binary executable
```bash
chmod +x xode-node
```

## Step 4: Create a base path directory
```bash
mkdir xode
```

## Step 5: Create a startup script

Create `xode-node.sh`:

```bash
#!/bin/bash

/home/ubuntu/xode-node \
--chain /home/ubuntu/raw-xode-node-chainspec.json \
--base-path /home/ubuntu/xode \
--rpc-port 9944 \
--pruning archive \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--name "xode-node"
```

### Description of flags

- `/home/ubuntu/xode-node`: path to the node binary
- `--chain /home/ubuntu/raw-xode-node-chainspec.json`: chain specification file path
- `--base-path /home/ubuntu/xode`: database directory
- `--rpc-port 9944`: HTTP/WebSocket RPC port
- `--pruning archive`: run in archive mode (omit for full node)
- `--telemetry-url "wss://telemetry.polkadot.io/submit/ 0"`: enable telemetry
- `--name "your-node-name"`: custom node name for telemetry

### Enable RPC (optional)

Add the following lines to expose RPC:

```bash
--rpc-external \
--unsafe-rpc-external \
--rpc-methods safe \
--rpc-cors '*'
```

## Full example with RPC enabled (archive)

```bash
#!/bin/bash

/home/ubuntu/xode-node \
--chain /home/ubuntu/raw-xode-node-chainspec.json \
--base-path /home/ubuntu/xode \
--rpc-port 9944 \
--pruning archive \
--rpc-external \
--unsafe-rpc-external \
--rpc-methods safe \
--rpc-cors '*' \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--name "xode-node"
```

## Step 6: Make the script executable
```bash
chmod +x xode-node.sh
```

## Step 7: Test run
```bash
./xode-node.sh
```

## Step 8: Create a systemd service

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

## Step 9: Enable the service
```bash
sudo systemctl enable xode-collator.service
```

## Step 10: Start the service
```bash
sudo systemctl start xode-collator.service
```

## Step 11: Check service logs
```bash
journalctl -f -u xode-collator | cat
```

## Step 12: Verify on telemetry

`https://telemetry.polkadot.io/#/0x28cc1df52619f4edd9f0389a7e910a636276075ecc429600f1dd434e281a04e9`


