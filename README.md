# 🚀 SSH Debug Action

**Debug GitHub runners on Windows, Linux & macOS via SSH with jump host tunneling, direct connections (e.g., Tailscale),
and remote access capabilities**

*Greatly inspired by [mxschmitt/action-tmate](https://github.com/mxschmitt/action-tmate) but takes a completely
different approach by installing an OpenSSH server on the runner, enabling standard SSH connections with full port
forwarding, VS Code remote development support, and flexible connectivity through your own jump hosts, public free
services (serveo.net, ssh-j.com, pinggy.io), or direct connections (e.g., Tailscale).*

---

## ✨ Features

- 🖥️ **Multi-Platform Support** - Works on Linux, Windows, and macOS runners
- 🔧 **OpenSSH Server** - Installs and configures OpenSSH server on the runner for standard SSH access
- 🌉 **Flexible Connectivity** - Use your own jump hosts, public free services (serveo.net, ssh-j.com, pinggy.io), or direct
  connections (e.g., Tailscale)
- 🔐 **Flexible Authentication** - Support for SSH keys, authorized keys, and GitHub actor keys
- ⚡ **Detached Mode** - Run SSH sessions in background while continuing workflow
- 🔄 **Session Management** - Configurable timeouts and termination controls
- 📖 **Rich Help Messages** - Beautiful connection guides with VS Code integration tips
- 🛠️ **Developer Friendly** - Perfect for debugging CI/CD issues and testing
- 💻 **VS Code** - Full remote development support with port forwarding
- ⚠️ **Public Runner Focused** - Designed for GitHub-hosted runners in isolated environments

---

## 🚀 Quick Start

### Basic Usage

Set up the following environment variables in your GitHub repository settings (Settings → Secrets and variables →
Actions → Variables):

```yaml
name: Debug Workflow
on: workflow_dispatch

# Define environment variables in GitHub UI (Repository Settings → Variables)
env:
  SSH_JUMP_HOST: 'your-jump-host.net'
  SSH_JUMP_PORT: '22'
  SSH_JUMP_USER: 'your-user'
  SSH_JUMP_FORWARD: '2222'
  SSH_JUMP_HOST_KEYS: |
    your-jump-host.net ssh-rsa AAAAB3NzaC1yc2E...

jobs:
  debug:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup SSH Debug Session
        uses: lexbritvin/ssh-debug-action@v1
        with:
          ssh-jump-host: ${{ env.SSH_JUMP_HOST }}
          ssh-jump-port: ${{ env.SSH_JUMP_PORT }}
          ssh-jump-user: ${{ env.SSH_JUMP_USER }}
          ssh-jump-host-keys: ${{ env.SSH_JUMP_HOST_KEYS }}
          ssh-jump-forward: ${{ env.SSH_JUMP_FORWARD }}
          ssh-jump-private-key: ${{ secrets.SSH_PRIVATE_KEY }} # Add to GitHub Secrets (not needed for public hosts)
          authorized-keys: ${{ secrets.SSH_PUBLIC_KEYS }}      # Add to GitHub Secrets
          use-actor-ssh-keys: 'true'
```

Check the Runner output for connection instructions.

### Public Jump Host Examples

Public jump hosts don't require a private key to connect and are great for quick debugging sessions.

#### ssh-j.com

```yaml
env:
  SSH_JUMP_HOST: 'ssh-j.com'
  SSH_JUMP_USER: ':generate' # Generates a scoped user for host alias
  SSH_JUMP_FORWARD: ':generate' # Generates a unique host alias within the jump host
  SSH_JUMP_HOST_KEYS: |
    ssh-j.com ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC...
```

#### pinggy.io

```yaml
env:
  SSH_JUMP_HOST: 'free.pinggy.io'
  SSH_JUMP_PORT: '443'
  SSH_JUMP_USER: 'tcp'  # Use 'tcp' to forward any TCP connection
  SSH_JUMP_FORWARD: '0' # Use random port allocation
  SSH_JUMP_HOST_KEYS: |
    [free.pinggy.io]:443 ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC...
```

#### serveo.net

```yaml
env:
  SSH_JUMP_HOST: 'serveo.net'
  SSH_JUMP_PORT: '22'
  SSH_JUMP_FORWARD: ':generate' # Generates a unique host alias (can also use '0' for random public port)
  SSH_JUMP_HOST_KEYS: |
    serveo.net ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC...
```

### 🎨 VS Code Remote Development

#### Standard SSH Connection

1. Install the **Remote - SSH** extension in VS Code
2. Open Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P` on macOS)
3. Select **"Remote-SSH: Connect to Host..."**
4. Enter the SSH command from the action outputs
5. Open the workspace directory on the remote host
6. Use VS Code's integrated terminal and port forwarding features

### 🔧 Port Forwarding

```bash
# Forward local port 3000 to remote port 8080
ssh -L 3000:localhost:8080 user@hostname -p 2222 -N -T

# Access forwarded service
curl http://localhost:3000

# Multiple port forwards
ssh -L 3000:localhost:8080 -L 5000:localhost:5000 user@hostname -p 2222
```

### Session Termination Methods

```bash
# Method 1: Create finish file (Linux/macOS)
touch ./finish-debug

# Method 2: PowerShell (Windows)
New-Item -ItemType File -Name './finish-debug' -Force

# Method 3: Cancel workflow in GitHub UI (Actions tab → Cancel)
```

---

## 🛡️ Security Considerations

- 🚫 **Public Access**: Be cautious with public jump host services - they may expose your session to others
- 🔐 **Private Keys**: Always store SSH private keys in GitHub Secrets, never in code
- 🔒 **Authorized Keys**: Limit authorized keys to trusted public keys only
- ⏱️ **Session Timeouts**: Set appropriate timeout values to prevent long-running sessions
- 🌐 **Network Access**: Consider using private networks like Tailscale for enhanced security
- ⚠️ **Self-Hosted Runners**: Use with caution on self-hosted runners - this action is intended for GitHub-hosted
  runners that start in clean, isolated environments

---

## 📋 Input Parameters

### 🔧 SSH Server Configuration

| Parameter                    | Description                                              | Default    | Required |
|------------------------------|----------------------------------------------------------|------------|----------|
| `ssh-server-host`            | SSH server hostname or IP address for client connections | `''`       | No       |
| `ssh-server-port`            | SSH server port                                          | `2222`     | No       |
| `ssh-server-user`            | SSH username (use `:current` for current user)           | `:current` | No       |
| `ssh-server-authorized-keys` | Authorized public keys (one per line)                    | `''`       | No       |

### 🌉 Jump Host Configuration

| Parameter                   | Description                                                    | Default | Required |
|-----------------------------|----------------------------------------------------------------|---------|----------|
| `ssh-jump-host`             | SSH jump host server                                           | `''`    | No       |
| `ssh-jump-port`             | SSH jump host port                                             | `22`    | No       |
| `ssh-jump-user`             | Jump host username (`:generate` for auto-generation)           | `''`    | No       |
| `ssh-jump-forward`          | Port forwarding config (`:generate`, `0`, `port`, `host:port`) | `''`    | No       |
| `ssh-jump-private-key`      | Private key content for jump host authentication               | `''`    | No       |
| `ssh-jump-private-key-path` | Private key file path for jump host authentication             | `''`    | No       |
| `ssh-jump-host-keys`        | SSH host keys for server verification                          | `''`    | No       |
| `ssh-jump-extra-flags`      | Additional SSH flags for jump host connection                  | `''`    | No       |

### 🔐 Authentication & Security

| Parameter            | Description                                   | Default | Required |
|----------------------|-----------------------------------------------|---------|----------|
| `use-actor-ssh-keys` | Authorize the triggering user's SSH keys      | `false` | No       |
| `authorized-keys`    | Additional authorized public keys (multiline) | `''`    | No       |

### ⏱️ Session Management

| Parameter              | Description                            | Default          | Required |
|------------------------|----------------------------------------|------------------|----------|
| `wait-for-file`        | File path to monitor for session end   | `./finish-debug` | No       |
| `wait-timeout`         | Maximum session duration (seconds)     | `1800`           | No       |
| `detached`             | Run SSH session in background          | `false`          | No       |
| `display-help-message` | Show connection instructions in output | `true`           | No       |

---

## 📤 Output Parameters

| Output          | Description                           |
|-----------------|---------------------------------------|
| `ssh-host`      | SSH server hostname/IP address        |
| `ssh-port`      | SSH server port number                |
| `ssh-user`      | SSH username for connection           |
| `ssh-jump-host` | Jump host hostname                    |
| `ssh-jump-port` | Jump host port number                 |
| `ssh-jump-user` | Jump host username                    |
| `ssh-host-keys` | SSH server host keys for verification |
| `ssh-command`   | Complete SSH connection command       |
| `help-message`  | Formatted connection guide            |

---

## 💡 Usage Examples

### 🐛 Debug Failed Tests

```yaml
name: Debug on Failure
on: [ push, pull_request ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Tests
        run: npm test
        continue-on-error: true

      - name: Setup Debug Session on Failure
        if: failure()
        uses: lexbritvin/ssh-debug-action@v1
        with:
          ssh-jump-host: ${{ env.SSH_JUMP_HOST }}
          ssh-jump-port: ${{ env.SSH_JUMP_PORT }}
          ssh-jump-user: ${{ env.SSH_JUMP_USER }}
          ssh-jump-host-keys: ${{ env.SSH_JUMP_HOST_KEYS }}
          ssh-jump-forward: ${{ env.SSH_JUMP_FORWARD }}
          ssh-jump-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
          authorized-keys: ${{ secrets.SSH_PUBLIC_KEYS }}
          use-actor-ssh-keys: 'true'
```

### 🔄 Detached Mode with Notifications

```yaml
- name: Start SSH Debug (Background)
  id: ssh-debug
  uses: lexbritvin/ssh-debug-action@v1
  with:
    ssh-jump-host: ${{ env.SSH_JUMP_HOST }}
    ssh-jump-port: ${{ env.SSH_JUMP_PORT }}
    ssh-jump-user: ${{ env.SSH_JUMP_USER }}
    ssh-jump-host-keys: ${{ env.SSH_JUMP_HOST_KEYS }}
    ssh-jump-forward: ${{ env.SSH_JUMP_FORWARD }}
    ssh-jump-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    authorized-keys: ${{ secrets.SSH_PUBLIC_KEYS }}
    use-actor-ssh-keys: 'true'
    display-help-message: 'false'
    detached: true

- name: Send Slack Notification
  uses: 8398a7/action-slack@v3
  with:
    status: custom
    custom_payload: |
      {
        "text": "🚀 SSH Debug Session Active",
        "blocks": [
          {
            "type": "section",
            "text": {
              "type": "mrkdwn",
              "text": "```${{ steps.ssh-debug.outputs.ssh-command }}```"
            }
          }
        ]
      }

- name: Display Connection Help
  env:
    HELP_MESSAGE: ${{ steps.ssh-debug.outputs.help-message }}
    EXTRA_HELP: |
      ╔══════════════════════════════════════════════════════════════════════════════════════════╗
      ║                              🐛 GO DEBUGGING WITH DELVE                                  ║
      ╚══════════════════════════════════════════════════════════════════════════════════════════╝
    
      \033[1;32m┌─ 🔄 PORT FORWARDING FOR DELVE\033[0m
      \033[1;32m│\033[0m   \033[1;97mssh -L 2345:localhost:2345 ...\033[0m
      \033[1;32m└─\033[0m
    
      \033[1;33m┌─ 🛠️  DEBUG COMMANDS\033[0m
      \033[1;33m│\033[0m   \033[1;97mRun all tests:\033[0m
      \033[1;33m│\033[0m     \033[97mgo test ./...\033[0m
      \033[1;33m│\033[0m   \033[1;97mRun specific test:\033[0m
      \033[1;33m│\033[0m     \033[97mgo test -v -run TestFooBar ./...\033[0m
      \033[1;33m│\033[0m   \033[1;97mDebug specific test:\033[0m
      \033[1;33m│\033[0m     \033[97mdlv --listen=:2345 --headless --api-version=2 test ./... -- -test.run TestFooBar\033[0m
      \033[1;33m│\033[0m   \033[1;97mDebug application:\033[0m
      \033[1;33m│\033[0m     \033[97mdlv debug --headless --listen=:2345 --api-version=2 ./cmd/foo -- [args...]\033[0m
      \033[1;33m└─\033[0m
    
      \033[1;34m┌─ 🔗 IDE INTEGRATION\033[0m
      \033[1;34m│\033[0m   \033[1;36mConnect to Delve on port 2345:\033[0m
      \033[1;34m│\033[0m   \033[1;36m•\033[0m \033[1;97mVS Code:\033[0m https://github.com/golang/vscode-go/blob/master/docs/debugging.md
      \033[1;34m│\033[0m   \033[1;36m•\033[0m \033[1;97mGoLand/IntelliJ:\033[0m https://www.jetbrains.com/help/go/attach-to-running-go-processes-with-debugger.html#attach-to-a-process-on-a-remote-machine
      \033[1;34m└─\033[0m
    
      \033[1;36m📚 Delve Documentation:\033[0m https://github.com/go-delve/delve/tree/master/Documentation
  run: |
    echo "SSH Debug Session Started!"
    # The help-message output contains ANSI color codes (\033[1;32m etc.)
    # Use printf to properly display colored output
    printf "%b\n%b\n" "$HELP_MESSAGE" "$EXTRA_HELP"
```

### 🌐 Tailscale Integration

Connect directly to your GitHub runners through your private Tailscale network for secure, NAT-traversal-free access
without requiring jump hosts.

#### Setting up Tailscale OAuth

1. Go to [Tailscale Admin Console](https://login.tailscale.com/admin/settings/oauth)
2. Generate OAuth credentials for GitHub Actions with appropriate scopes
3. Configure ACL tags for CI runners (recommended: `tag:ci`)
4. Add the following to your GitHub repository secrets:
    - `TS_OAUTH_CLIENT_ID` - Your Tailscale OAuth client ID
    - `TS_OAUTH_SECRET` - Your Tailscale OAuth secret

#### Tailscale ACL Configuration

Add this to your Tailscale ACL policy to allow SSH access to CI runners:

```json
{
  "tagOwners": {
    "tag:ci": [
      "your-email@example.com"
    ]
  },
  "acls": [
    {
      "action": "accept",
      "src": [
        "your-email@example.com"
      ],
      "dst": [
        "tag:ci:22"
      ]
    }
  ]
}
```

#### Workflow Example

```yaml
name: Debug with Tailscale
on: workflow_dispatch

jobs:
  debug:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Connect to Tailscale
        uses: tailscale/github-action@v2
        with:
          oauth-client-id: ${{ secrets.TS_OAUTH_CLIENT_ID }}
          oauth-secret: ${{ secrets.TS_OAUTH_SECRET }}
          tags: tag:ci

      - name: Get Tailscale IP
        id: tailscale-ip
        run: |
          TAILSCALE_IP=$(tailscale ip -4)
          echo "ip=$TAILSCALE_IP" >> $GITHUB_OUTPUT
          echo "🌐 Runner accessible at: $TAILSCALE_IP"

      - name: Setup SSH Debug (Tailscale Direct)
        uses: lexbritvin/ssh-debug-action@v1
        with:
          # No jump host needed - direct connection via Tailscale
          ssh-server-host: ${{ steps.tailscale-ip.outputs.ip }}
          ssh-server-port: 2223
          use-actor-ssh-keys: true
          wait-timeout: 3600
```

---

## 🔧 Advanced Configuration

### Environment Variables Setup

#### Repository Variables (Settings → Secrets and variables → Actions → Variables)

These are non-sensitive configuration values that can be shared:

- `SSH_JUMP_HOST` - Your jump host hostname
- `SSH_JUMP_PORT` - Jump host SSH port (usually 22)
- `SSH_JUMP_USER` - Username for jump host connection
- `SSH_JUMP_FORWARD` - Port forwarding configuration
- `SSH_JUMP_HOST_KEYS` - SSH host keys for verification

#### Repository Secrets (Settings → Secrets and variables → Actions → Secrets)

These are sensitive values that should be encrypted:

- `SSH_PRIVATE_KEY` - Private key for jump host authentication
- `SSH_PUBLIC_KEYS` - Authorized public keys for runner access
- `TS_OAUTH_CLIENT_ID` - Tailscale OAuth client ID (if using Tailscale)
- `TS_OAUTH_SECRET` - Tailscale OAuth secret (if using Tailscale)

### Custom SSH Configurations

#### Skipping Host Key Verification

```yaml
- name: SSH Debug with Skipped Host Check
  uses: lexbritvin/ssh-debug-action@v1
  with:
    ssh-jump-host: ${{ env.SSH_JUMP_HOST }}
    # Do not check a server key and use verbose SSH output
    ssh-jump-extra-flags: "-o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -vvv"
    use-actor-ssh-keys: true
```

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

Built with ❤️ for the GitHub Actions community