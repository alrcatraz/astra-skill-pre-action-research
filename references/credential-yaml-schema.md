# 基础设施凭证 YAML Schema

> 分组存储：`~/Documents/credentials/{personal,work,other}-credentials.yaml.gpg`
> 临时凭证不持久化。
> 加密密钥：Alrcatraz <alrcatraz@gmx.com> (GPG), 密码见 `.env` → `GPG_Key_Alrcatraz`
> `README.md` 仅为索引，不存实际值。

---

## 完整结构

```yaml
devices:
  <identifier>:
    hostname: "<hostname>"
    os:
      name: "<distro name>"
      kernel: "<kernel version>"         # optional
      version: "<os version>"            # optional
    description: "<one-line purpose>"
    tags:
      - <personal|work|other>
      - <laptop|desktop|server|nas|router|embedded>
    network:
      main_ip: "<primary IP>"
      interfaces:                        # optional
        - name: "<eth0|wlan0>"
          mac: "<MAC>"                   # port identity only, NOT device identity
          purpose: "<wired|wifi|management>"
      overlay:
        easy_tier: "<IP>"
        zero_tier: "<IP>"
        tailscale: "<IP>"
    connection:
      paths:                             # ordered by priority (1 = best)
        - priority: 1
          type: "direct"                 # direct / jump / proxy
          via: []                        # empty for direct
          note: "<description>"
        - priority: 2
          type: "jump"
          via:
            - host: "<jump host id>"
              user: "<username>"
              ip: "<jump IP>"
            - host: "<second hop>"
              user: "<username>"
              ip: "<second hop IP>"
          note: "<description>"
    roles:
      - jump_host
      - proxy_node
      - storage
      - compute_server
      - gateway
    access:
      methods:
        - type: ssh_key
          key_path: "<~/.ssh/id_xxx>"
          note: "deployed ✅"
        - type: password
          value: "<password>"
          note: "fallback when ssh_key unavailable"
    accounts:
      - username: "<username>"
        is_admin: true                   # true / false
        access: sudo                     # sudo / su / none
        note: "<description>"
    services:                            # optional
      - name: "<service name>"
        type: proxy|web_ui|api|mcp
        port: <port>
        url: "<URL>"                    # optional
        accounts:
          - username: "<username>"
            password: "<password>"
            note: "<description>"
```

## Headless GPG Access Pattern (GPG 2.5+)

```bash
# Store passphrase in .env:
#   export GPG_Key_Alrcatraz=your_passphrase

# Decrypt (safe — no args, no terminal echo):
source <(grep 'GPG_Key_Alrcatraz' ~/.hermes/.env)
echo "$GPG_Key_Alrcatraz" | gpg --batch --no-tty --passphrase-fd 0 \
  --pinentry-mode loopback --decrypt <file> 2>/dev/null | grep <key>

# Encrypt:
cat <content> | gpg --batch --yes --symmetric \
  --passphrase "$GPG_Key_Alrcatraz" --cipher-algo AES256 \
  --output ~/Documents/credentials/<group>-credentials.yaml.gpg
```

> **`--pinentry-mode loopback` is REQUIRED.** GPG 2.5+ drops the agent check in batch mode when this flag is present. Without it, `gpg-agent` hangs waiting for a GUI pinentry that never arrives in a headless session.

## Design Principles

| Principle | Rationale |
|:----------|:----------|
| **Path-first** | `connection.paths[]` ordered by priority — direct, single-hop, multi-hop. All supported uniformly. |
| **Accounts separate from access methods** | `accounts[]` = who has what permissions; `access.methods[]` = how to connect. Two orthogonal concerns. |
| **Service-level nesting** | Management interfaces (BMC web UI, DSM) get their own accounts under the service section, not mixed with OS accounts. |
| **MAC = port identity only** | MAC changes with NIC replacement. Never use MAC to identify a device. |
| **Passwords can be null** | `null` means "pending confirmation" — leaves a trace without misleading. |

## Group Files

| File | Contains | Example Devices |
|:-----|:---------|:----------------|
| `personal-credentials.yaml.gpg` | Personal devices | SUSET01, DS425Plus, OpenWrt, FedoraTG |
| `work-credentials.yaml.gpg` | Work devices | GPU server, BMC, customer environments |
| `other-credentials.yaml.gpg` | Friends/family | Shared devices, guest access |
| Not stored | Temporary credentials | One-time use, discarded immediately |

## GUID (Group Unique IDentifier) — `access.methods[].guid`

Each access method MAY carry a `guid` field that acts as a stable, collision-free identifier for a specific authentication pathway (a specific SSH key, a specific password entry in a GPG file, a specific API token, etc.).

### Format

`<scheme>:<unique_value>`, where `<scheme>` is a well-known prefix:

| Scheme | Example | Purpose |
|--------|---------|---------|
| `key` | `key:suset01-id-ed25519` | Identifies a specific SSH key on disk |
| `pwd` | `pwd:suset01-alrcatraz` | Identifies a specific password entry in a GPG file |
| `token` | `token:hermes-gateway-matrix` | Identifies a specific API token |

### How GUIDs connect the credential file and device inventory

**In the GPG credential file** (`personal-credentials.yaml.gpg`):

```yaml
devices:
  suset01:
    access:
      methods:
        - type: ssh_key
          guid: "key:suset01-id-ed25519"          # ← GUID
          key_path: "~/.ssh/id_ed25519_jump_star"
          note: "primary key, deployed 2026-06-07"

        - type: password
          guid: "pwd:suset01-alrcatraz"            # ← GUID
          value: "401503"
          note: "SSH fallback, same as sudo"
```

**In the device inventory skill** (`infrastructure-device-inventory`):

```yaml
devices:
  suset01:
    hostname: "SUSET01"
    access_guids:                                  # ← References only
      - "key:suset01-id-ed25519"
      - "pwd:suset01-alrcatraz"
```

### Why GUIDs solve a real problem

| Without GUID | With GUID |
|:-------------|:----------|
| Device inventory says "SSH key ✅" — no way to know *which* key | Inventory says `guid: key:suset01-id-ed25519` — cross-reference credential file to find the exact key path |
| Credential file says "password changed" — no way to link to device entries that reference it | Same GUID in both places — a change in the credential file is immediately traceable to all devices using that credential |
| After a credential rotation, you must manually find every device that used the old one | A search for `guid: pwd:suset01-alrcatraz` surfaces every reference immediately |

### Scope

GUIDs are a **value-add enhancement** to the schema — they are NOT required. A credential file without GUIDs is valid and functional. Add them when an inventory spans dozens of devices and credential rotation traceability matters.
