---
name: frodo
description: "frodo CLI (v3.1.0) for managing PingOne Advanced Identity Cloud (AIC). Handles config export/import, journeys/trees, SAML providers, scripts, OAuth2 clients, ESV secrets/variables, log tailing, mappings, themes, services, agents, authorization policies, and tenant promotion. Contains PAIC team environment names, standard flag combos, and workflow recipes."
---

# Frodo CLI — PAIC Team Skill

For full flag details, run `frodo <command> <action> --help`.

- CLI repo: https://github.com/rockcarver/frodo-cli
- Library repo: https://github.com/rockcarver/frodo-lib

## Environments

Use these shorthand substrings with any frodo command (matches against saved connection profiles):

| Shorthand | Tenant | Purpose |
|-----------|--------|---------|
| `sb2` | commkentsb2 | Sandbox 2 |
| `sb3` | commkentsb3 | Sandbox 3 |
| `wf` | commkentwf | Development |
| `uat2` | commkentwf-uat2 | UAT 2 |
| `cidmhcsc` | cidmhcsc | CIDM HCSC dev |

```bash
frodo conn list                  # See all saved profiles
frodo conn describe sb2          # Describe a specific profile
```

## Config Export/Import

**Syntax**: `frodo config export [options] [host] [realm] [username] [password]`

```bash
# Team standard — snapshot to dated folder
frodo config export -sxoAND ./sb2_20260227 sb2

# Everything to single file
frodo config export -a sb2

# Specific realm only
frodo config export -ar sb2 bravo

# With encrypted secret values for cross-env migration
frodo config export -a --include-active-values --target sb3 sb2
```

**Syntax**: `frodo config import [options] [host] [realm] [username] [password]`

```bash
# Import from separate files
frodo config import -A -D ./sb2_export sb3

# Import from single file
frodo config import -a -f sb2-all.json sb3
```

Note: `-g` for global-only, `-r` for realm-only, `-R` for read-only config, `-d` to include default scripts.

## Journey/Tree

**Syntax**: `frodo journey <list|export|import|delete|describe|disable|enable|prune> [options] [host] [realm]`

```bash
frodo journey list sb2
frodo journey export -i "MyLoginJourney" sb2               # Single with deps
frodo journey export -i "MyLoginJourney" --no-deps sb2     # Single without deps
frodo journey export -A sb2                                 # All to separate files
frodo journey import -i "MyLoginJourney" -f MyLoginJourney.journey.json sb3
frodo journey import -A -D ./journeys sb3
frodo journey delete -i "MyLoginJourney" sb2
frodo journey describe -i "MyLoginJourney" sb2
frodo journey disable -i "MyLoginJourney" sb2
frodo journey enable -i "MyLoginJourney" sb2
frodo journey prune sb2                                     # Clean orphaned artifacts
```

Note: `--no-deps` skips scripts, SAML providers, themes, social IdPs. `--no-coords` skips node positions.

## SAML

**Syntax**: `frodo saml <list|export|import|delete|describe> [options] [host] [realm]`

```bash
frodo saml list sb2
frodo saml export -A sb2
frodo saml export -i "https://sp.example.com/saml" sb2
frodo saml import -i "https://sp.example.com/saml" -f provider.saml.json sb3
frodo saml delete -i "https://sp.example.com/saml" sb2
frodo saml describe -i "https://sp.example.com/saml" sb2
```

Related — Circles of Trust and metadata:

```bash
frodo saml cot list sb2
frodo saml cot export -A sb2
frodo saml cot import -A sb3
frodo saml metadata export -i "https://sp.example.com/saml" sb2
```

## Script

**Syntax**: `frodo script <list|export|import|delete|describe> [options] [host] [realm]`

```bash
frodo script list sb2
frodo script export -A sb2                    # All to separate files
frodo script export -n "MyScript" sb2         # By name
frodo script export -i <script-uuid> sb2      # By UUID
frodo script import -A sb3
frodo script delete -i <script-uuid> sb2
frodo script describe -i <script-uuid> sb2
```

## OAuth2 Client

**Syntax**: `frodo oauth client <list|export|import|delete> [options] [host] [realm]`

```bash
frodo oauth client list sb2
frodo oauth client export -A sb2
frodo oauth client export -i "myClientId" sb2
frodo oauth client import -A sb3
frodo oauth client delete -i "myClientId" sb2
```

## ESV (Secrets & Variables)

**Syntax**: `frodo esv <secret|variable> <list|describe|export|import|set> [options] [host]`

```bash
frodo esv secret list -lu sb2                            # List with usage info
frodo esv secret describe -i "esv-my-secret" sb2
frodo esv variable list sb2
frodo esv variable describe -i "esv-my-variable" sb2
frodo esv apply sb2                                      # Apply pending changes
```

Note: `esv apply` triggers pod restart, takes up to 10 minutes. `-l` for long format, `-u` for usage info.

## Log

**Syntax**: `frodo log <tail|fetch|list> [options] [host]`

```bash
frodo log tail sb2                         # All logs, all levels
frodo log tail -l 0 sb2                    # Errors only (0=ERROR, 1=WARN, 2=INFO, 3=DEBUG, 4=ALL)
frodo log tail -c am-core,idm-core sb2     # Specific sources
frodo log tail -t <txid> sb2               # Filter by transaction ID
frodo log tail -d sb2                      # Default noise filters
frodo log fetch sb2                        # Historical logs
frodo log list sb2                         # Available log sources
```

## Other Resources (Same Pattern)

All follow: `frodo <resource> <list|export|import|delete> [options] [host] [realm]`

Consistent flags: `-a` (all/single file), `-A` (all/separate files), `-i` (specific item), `-f` (file), `-D` (directory).

```bash
# Applications (NOT OAuth2 clients — use frodo oauth client for those)
frodo app list sb2
frodo app export -A sb2
frodo app import -A sb3

# Email templates (delete new in v3.1.0)
frodo email template list sb2
frodo email template export -A sb2
frodo email template delete -i <template-id> sb2

# IDM mappings
frodo mapping list sb2
frodo mapping export -A sb2
frodo mapping rename sb2             # Legacy (sync/) → new (mapping/) naming
frodo mapping rename -l sb2          # New → legacy

# IDM config objects
frodo idm list sb2
frodo idm export -A sb2
frodo idm count -n alpha_user sb2

# Social identity providers, AM services, themes, realms, roles, agents, authn, authz
frodo idp list sb2
frodo service list sb2
frodo theme list sb2
frodo realm list sb2
frodo realm describe sb2
frodo role list sb2
frodo agent list sb2
frodo authn describe sb2
frodo authz policy list sb2
frodo authz set list sb2
frodo authz type list sb2
```

## Admin

**Syntax**: `frodo admin <command> [options] [host]`

```bash
frodo admin list-oauth2-clients-with-admin-privileges sb2
frodo admin grant-oauth2-client-admin-privileges sb2
frodo admin revoke-oauth2-client-admin-privileges sb2
frodo admin list-static-user-mappings sb2
frodo admin repair-org-model sb2
frodo admin federation list sb2
frodo info sb2                                    # Tenant version/connection info
frodo info --json sb2
```

## Gotchas

- `frodo app` manages AIC application templates (v2+), NOT OAuth2 clients. Use `frodo oauth client` for OAuth2.
- `frodo config help export` works (config has a special `help` subcommand with examples), but other commands need `frodo <cmd> <action> --help`.
- Connection profiles use URL substring matching — `sb2` works because it uniquely matches the sb2 tenant URL.
- `esv apply` triggers a pod restart that takes up to 10 minutes. Don't re-run or assume failure if slow.
- Export flags `-a`/`-A` in v3.0.8+ no longer stop prematurely on errors.
- `email template delete` is new in v3.1.0.
- Proxy Connect support (`--authentication-header-overrides` on `conn save`) added in v3.0.10.
