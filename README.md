# arm64-task-scheduler
Commands for running a .NET/Python task scheduler in OracleCloud VM w/ARM64.


## Connect to VM via SSH
```bash
ssh -i "C:\[...]\SSHPrivate.key" ubuntu@[VM PUBLIC IP]
```

---

## Manual installation steps for .NET8 on ARM64 architecture

### Prepare script
```bash
cd ~
curl -L https://dot.net/v1/dotnet-install.sh -o dotnet-install.sh
chmod +x dotnet-install.sh
```

### Install
```bash
./dotnet-install.sh --channel 8.0 --architecture arm64 --install-dir "$HOME/.dotnet"
```

### Add .NET to PATH
```bash
echo 'export DOTNET_ROOT=$HOME/.dotnet' >> ~/.bashrc
echo 'export PATH=$PATH:$DOTNET_ROOT:$DOTNET_ROOT/tools' >> ~/.bashrc
source ~/.bashrc
```

### Verify
```bash
dotnet --version
dotnet --info
```

```bash
cd ~/homelab/dotnet
mkdir test
cd test
dotnet new console
dotnet run
```

```text
Hello, World!
```

---

## Configure a scheduled service to execute a batch

### Prepare a Test Console in .NET8

```bash
cd ~/homelab/dotnet/apps
mkdir test-batch
cd test-batch
dotnet new console
nano Program.cs
```

```cs
using System;

Console.WriteLine($"Batch executed: {DateTime.Now:yyyy-MM-dd HH:mm:ss}");
Console.WriteLine($"Server: {Environment.MachineName}");
Console.WriteLine($"OS: {Environment.OSVersion}");
```

```bash
dotnet run
```

```text
Batch executed: 2026-09-21 14:xx:xx
Server: xxx-xxx-xxx
OS: Unix ...
```

### Publish an executable .dll (_to be deployed on VM via SCP - detailed later_)
```bash
dotnet publish -c Release -r linux-arm64 --self-contained false -o ~/homelab/dotnet/apps/test-batch/publish
```

### Check
```bash
ls -lh ~/homelab/dotnet/apps/test-batch/publish
```

### Execute the .dll manually
```bash
dotnet ~/homelab/dotnet/apps/test-batch/publish/test-batch.dll
```

---

### Prepare a script to execute the .dll

```bash
nano ~/homelab/scripts/test-batch.sh
```

```bash
#!/bin/bash

echo "===== START $(date '+%Y-%m-%d %H:%M:%S') ====="

dotnet "$HOME/homelab/dotnet/apps/test-batch/publish/test-batch.dll"

EXIT_CODE=$?

echo "===== END $(date '+%Y-%m-%d %H:%M:%S') - EXIT CODE: $EXIT_CODE ====="

exit $EXIT_CODE
```

### Make the script executable
```bash
chmod +x ~/homelab/scripts/test-batch.sh
```

### Execute the script manually 
_(note: the configuration paths like "../" are relative to the current folder. If needed, "cd" to the project folder where the .dll is being run)_
```bash
~/homelab/scripts/test-batch.sh
```

```text
===== START 2026-09-21 14:22:39 =====
Batch eseguito: 2026-09-21 14:22:39
Server: etrlz-forge01-homelab
OS: Unix 7.0.0.1009
===== END 2026-09-21 14:22:39 - EXIT CODE: 0 =====
```

---

### Prepare a service to execute the script
```bash
sudo nano /etc/systemd/system/homelab-test-batch.service
```

```ini
[Unit]
Description=Homelab Test .NET Batch
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
User=ubuntu
WorkingDirectory=/home/ubuntu/homelab/dotnet/apps/test-batch
Environment="DOTNET_ROOT=/home/ubuntu/.dotnet"
Environment="PATH=/home/ubuntu/.dotnet:/home/ubuntu/.dotnet/tools:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
ExecStart=/home/ubuntu/homelab/scripts/test-batch.sh
```

```bash
sudo systemctl daemon-reload
```

### Execute the service manually
```bash
sudo systemctl start homelab-test-batch.service
```

### Check the status of the service / the output of the batch
```bash
sudo systemctl status homelab-test-batch.service
```
```bash
journalctl -u homelab-test-batch.service --no-pager
```

---
### Create a timer to execute the service
```bash
sudo nano /etc/systemd/system/homelab-test-batch.timer
```

```ini
[Unit]
Description=Run Homelab Test .NET Batch every 5 minutes

[Timer]
OnBootSec=1min
OnUnitActiveSec=5min
Unit=homelab-test-batch.service

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl daemon-reload
```
```bash
sudo systemctl enable --now homelab-test-batch.timer
```

### Check the specified timer
```bash
systemctl list-timers --all | grep homelab
```
_(use --all for all the system timers)_

### Check the timer executions
```bash
journalctl -u homelab-test-batch.service --no-pager
```

---

### Disable the timer
```bash
sudo systemctl disable --now homelab-test-batch.timer
```

---

## Build and Release

### Build the local project
```bash
dotnet publish -c Release -r linux-arm64 --self-contained false -o .\publish
```

### Transfer to server via SSH
```bash
scp -i "C:\[...]\SSHPrivate.key" -r ".\publish\*" ubuntu@[VM PUBLIC IP]:/home/ubuntu/homelab/dotnet/apps/MyBatch/
```

### Test manually
```bash
dotnet MyBatch.dll
```
