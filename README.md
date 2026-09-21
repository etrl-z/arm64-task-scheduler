# arm64-task-scheduler
Commands for running a .NET/Python task scheduler in OracleCloud VM w/ARM64.


**Connect to VM via SSH**
```bash
ssh -i "C:\[...]\SSHPrivate.key" [VM USER NAME]@[VM PUBLIC IP]
```

---

## Manual installation steps for .NET8 on ARM64 architecture:

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

### Test Batch in .NET8

```bash
cd ~/homelab/dotnet/apps
mkdir test-batch
cd test-batch
dotnet new console
nano Program.cs
```

```cs
using System;

Console.WriteLine($"Batch eseguito: {DateTime.Now:yyyy-MM-dd HH:mm:ss}");
Console.WriteLine($"Server: {Environment.MachineName}");
Console.WriteLine($"OS: {Environment.OSVersion}");
```

```bash
dotnet run
```

```text
Batch eseguito: 2026-09-21 14:xx:xx
Server: xxx-xxx-xxx
OS: Unix ...
```

### Publish an Executable .dll
```bash
dotnet publish -c Release -r linux-arm64 --self-contained false -o ~/homelab/dotnet/apps/test-batch/publish
```

### Check
```bash
ls -lh ~/homelab/dotnet/apps/test-batch/publish
```

### Execute manually
```bash
dotnet ~/homelab/dotnet/apps/test-batch/publish/test-batch.dll
```

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

### Make script Executable
```bash
chmod +x ~/homelab/scripts/test-batch.sh
```

### Execute the script
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


