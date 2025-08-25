---
sidebar_position: 7
title: Compiling XODE
description: Set up GitHub SSH, Rust toolchain, and build XODE for x86_64 and aarch64
---

# Compiling XODE

## Set up your GitHub environment

### Check existing SSH keys
```bash
ls -al ~/.ssh
```
Look for files like `id_rsa` and `id_rsa.pub`.

### Generate a new SSH key (if needed)
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### Add SSH key to the agent
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

### Copy the public key
```bash
cat ~/.ssh/id_rsa.pub
```

Add the key to GitHub: Settings → SSH and GPG keys → New SSH key.

### Clone XODE Blockchain
```bash
git clone git@github.com:Blockspace-Corporation/xode-blockchain.git
cd xode-blockchain
```

## Set up your Rust environment

### Install build tools
```bash
sudo apt install -y build-essential
```

### Install support packages
```bash
sudo apt install -y git clang curl libssl-dev llvm libudev-dev make protobuf-compiler
```

### Install Rust (rustup)
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
```

### Configure toolchains
```bash
rustup default stable
rustup update

rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
rustup default nightly-x86_64-unknown-linux-gnu

# Also ensure contracts WASM target is available
rustup target add wasm32-unknown-unknown
```

### Verify configuration
```bash
rustup show
rustup +nightly show
```

## Compile XODE
```bash
cargo build
```

## Cross-compiling for Raspberry Pi (aarch64)

### Install cross
```bash
cargo install cross
```

### Install Docker
```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
apt-cache policy docker-ce
sudo apt install -y docker-ce
sudo systemctl status docker
```

Allow Docker without sudo:
```bash
sudo usermod -aG docker ${USER}
su - ${USER}
groups
```

### Cross-compile
```bash
cross +nightly build --target aarch64-unknown-linux-gnu --release
```

## Running XODE with Zombienet

### Verify the built node
```bash
./target/release/xode-node --version
```

### Download Polkadot and Zombienet (from your zombienet directory)
```bash
wget https://github.com/paritytech/polkadot/releases/download/v1.0.0/polkadot
wget https://github.com/paritytech/zombienet/releases/download/v1.3.109/zombienet-linux-x64
chmod +x polkadot zombienet-linux-x64
```

### Launch XODE with Zombienet
```bash
./zombienet-launch.sh
```


