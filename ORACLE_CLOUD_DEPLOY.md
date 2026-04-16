# Oracle Cloud Free 24/7 Deployment Guide

This guide shows how to run your Telegram Instagram monitor bot on an Oracle Always Free VM.

## 1) Create Oracle account and VM

1. Sign up at [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/).
2. Open Oracle Console: `Compute -> Instances -> Create instance`.
3. Recommended settings:
   - Shape: `VM.Standard.E2.1.Micro` (Always Free x86)
   - Image: Ubuntu 22.04
4. Add your SSH public key.
5. Create the instance.

## 2) Network and firewall

- Keep SSH port `22` open.
- For Telegram polling bot mode, you do not need extra inbound ports.

## 3) Connect via SSH

```bash
ssh ubuntu@<YOUR_VM_PUBLIC_IP>
```

## 4) Install Docker and tools

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose-plugin git
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ubuntu
```

Disconnect and reconnect SSH so Docker group permissions apply.

## 5) Get your project onto the VM

If your code is on GitHub:

```bash
git clone <YOUR_REPO_URL>
cd the_watcher_V2.0
```

If not, upload with `scp` from your local machine.

## 6) Create environment file

```bash
cp .env.example .env
nano .env
```

Set these values:

- `TELEGRAM_ENABLED=true`
- `TELEGRAM_BOT_TOKEN=...`
- `TELEGRAM_CHAT_ID=...`
- `SESSION_USERNAME=...`
- `SESSION_PASSWORD=...` (if needed)

## 7) Build and run the bot container

```bash
docker build -t ig-telegram-bot .
docker run -d \
  --name ig-telegram-bot \
  --restart unless-stopped \
  --env-file .env \
  -v $HOME/ig-data:/data \
  ig-telegram-bot \
  <target_username> --telegram
```

Notes:

- `--restart unless-stopped` gives auto-restart after reboot/crash.
- `-v $HOME/ig-data:/data` persists runtime data.

## 8) Verify bot status

```bash
docker ps
docker logs -f ig-telegram-bot
```

In Telegram, test:

- `/help`
- `/status`
- `/targets`

## 9) Keep server secure and updated

- Use SSH keys only.
- Regular updates:

```bash
sudo apt update && sudo apt upgrade -y
```

## 10) Useful maintenance commands

Restart container:

```bash
docker restart ig-telegram-bot
```

Stop container:

```bash
docker stop ig-telegram-bot
```

Start container:

```bash
docker start ig-telegram-bot
```

Remove old image builds:

```bash
docker image prune -f
```
Yes — you can, and it works surprisingly well for small bots.

Short answer
Possible: yes, run your Telegram bots on unused Android.
Good for: hobby/small bots, lightweight workloads.
Not ideal for: critical 24/7 production reliability.
How people do it
Install Termux on the phone.
In Termux, install Python/Node + your bot dependencies.
Run bots with process manager (tmux, screen, or nohup).
Keep phone always plugged in, Wi‑Fi stable, battery optimization disabled for Termux.
Main limitations
Android may kill background tasks unless configured carefully.
Reboots/updates can stop processes.
Storage and thermal throttling can affect long runs.
Harder remote management than a VPS.
For your bot specifically
Your bot is polling Telegram and monitoring Instagram, so on Android you can run it, but you should:

use conservative check intervals,
keep persistent files in a dedicated folder,
auto-restart on boot (Termux:Boot app),
monitor logs regularly.
If you want, I can give you an exact Termux setup script for your current project (copy/paste commands).