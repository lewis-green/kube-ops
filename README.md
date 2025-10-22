# Kubernetes GitOps

Simplified Kubernetes cluster management with Talos Linux and Flux CD.

## Quick Start

### Prerequisites

- `talosctl` - Talos Linux CLI
- `talhelper` - Talos configuration helper
- `kubectl` - Kubernetes CLI
- `flux` - Flux CD CLI
- `op` - 1Password CLI
- `sops` - Secret encryption
- `task` - Task runner

### Bootstrap a New Cluster

1. **Configure your cluster** in `kubernetes/fsn1/bootstrap/talos/talconfig.yaml`

2. **Store secrets in 1Password**:
   ```bash
   # Generate and store age key for SOPS
   age-keygen | op document create --title "main-age-key" --vault Homelab -

   # Generate and store GitHub deploy key
   ssh-keygen -t ed25519 -C "flux@main" -f deploy_key
   op document create --title "main-github-deploy-key" --vault Homelab - < deploy_key
   ```

3. **Update .sops.yaml** with your age public key:
   ```bash
   op read "op://Homelab/main-age-key/public-key"
   ```

4. **Bootstrap Talos** (one command does it all):
   ```bash
   task talos:bootstrap controller=192.168.1.10
   ```

5. **Bootstrap Flux**:
   ```bash
   task flux:bootstrap
   ```

Done! Your cluster is now running with GitOps.

## Common Tasks

### Talos Management

```bash
# Bootstrap new cluster
task talos:bootstrap controller=192.168.1.10

# Open Talos dashboard
task talos:dashboard

# Upgrade Talos and Kubernetes
task talos:upgrade

# Fetch/refresh kubeconfig
task talos:kubeconfig
```

### Flux Management

```bash
# Bootstrap Flux
task flux:bootstrap

# Force reconcile all
task flux:reconcile

# Sync specific app
task flux:sync app=my-app

# Check Flux status
task flux:status

# Watch Flux logs
task flux:logs
```

### Kubernetes Operations

```bash
# List nodes
task k8s:nodes

# List all pods
task k8s:pods

# List pods in namespace
task k8s:pods ns=kube-system

# Get pod logs
task k8s:logs pod=my-pod ns=default

# Execute command in pod
task k8s:exec pod=my-pod ns=default cmd=/bin/bash

# Delete failed pods
task k8s:delete-failed

# Raw kubectl access
task k8s:shell -- get pods -A
```

### SOPS Operations

```bash
# Encrypt a file
task sops:encrypt file=secret.yaml

# Decrypt a file
task sops:decrypt file=secret.yaml

# Edit encrypted file
task sops:edit file=secret.yaml
```

## Directory Structure

```
.
├── .taskfiles/          # Modular task definitions
│   ├── flux/           # Flux CD tasks
│   ├── k8s/            # Kubernetes tasks
│   ├── sops/           # Secret management tasks
│   └── talos/          # Talos Linux tasks
├── kubernetes/
│   └── main/           # Main cluster
│       ├── bootstrap/  # Bootstrap configurations
│       │   ├── talos/       # Talos configs (talconfig.yaml, etc)
│       │   ├── flux/        # Flux installation manifests
│       │   └── integrations/ # CNI, CSI drivers, etc
│       ├── flux/       # Flux configuration
│       │   ├── config/      # Kustomizations
│       │   ├── vars/        # Cluster settings and secrets
│       │   └── repositories/ # Helm/Git sources
│       ├── apps/       # Application deployments
│       └── templates/  # Reusable templates
├── Taskfile.yaml       # Main task definitions
├── .sops.yaml          # SOPS encryption rules
└── README.md
```

## Secrets Management

This repo uses **1Password CLI + SOPS** for secrets:

- **1Password**: Stores bootstrap secrets (age keys, deploy keys)
- **SOPS**: Encrypts secrets in git using age encryption
- Secrets are automatically pulled from 1Password during bootstrap

### Adding New Secrets

1. Store in 1Password:
   ```bash
   echo "my-secret" | op document create --title "my-secret" --vault Homelab -
   ```

2. Reference in tasks or create SOPS-encrypted Kubernetes secrets:
   ```bash
   kubectl create secret generic my-secret --from-literal=key=value --dry-run=client -o yaml > secret.yaml
   task sops:encrypt file=secret.yaml
   ```

## Architecture Decisions

### Why This Structure?

1. **Simplified commands**: One command bootstraps entire cluster
2. **1Password integration**: No plain-text secrets in repo
3. **Modular tasks**: Each component (talos, flux, k8s) is separate
4. **Sensible defaults**: `cluster=main` is default, less typing
5. **GitOps ready**: Flux watches the repo for changes

### Improvements Over Previous Setup

- Fewer commands (bootstrap = 1 command vs 5+)
- Integrated secret management with 1Password
- Better organized taskfiles
- Clearer separation of concerns
- More automation, less manual steps

## Contributing

1. Make changes in feature branch
2. Test with `task --dry`
3. Run pre-commit: `pre-commit run --all-files`
4. Submit PR

## License

MIT
