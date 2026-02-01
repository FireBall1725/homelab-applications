# Secret Management with Sealed Secrets

This homelab uses **Sealed Secrets** to safely store sensitive data in git. Secrets are encrypted with a public key and can only be decrypted by the cluster.

## How It Works

1. **SealedSecret Controller** runs in your cluster (installed in `kube-system`)
2. You create a regular Kubernetes Secret locally
3. **kubeseal** encrypts it using the cluster's public key
4. The encrypted **SealedSecret** can be safely committed to git
5. When applied to the cluster, it's automatically decrypted into a regular Secret

## Quick Start

### Encrypting a New Secret

```bash
# Create a secret and encrypt it in one command
kubectl create secret generic <secret-name> \
  --namespace=<namespace> \
  --from-literal=<key>=<value> \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > <app-name>/templates/sealed-secret.yaml
```

### Example: Cribl Edge Token

```bash
kubectl create secret generic cribl-edge-leader \
  --namespace=app-cribl-edge \
  --from-literal=leader='tls://TOKEN@server.cribl.cloud?group=homelab' \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > cribl-edge/templates/sealed-secret.yaml
```

### Using the Secret in values.yaml

```yaml
<chart-name>:
  env:
    - name: MY_SECRET_VAR
      valueFrom:
        secretKeyRef:
          name: <secret-name>
          key: <key-name>
```

Or if the chart supports `existingSecret`:

```yaml
<chart-name>:
  existingSecret: <secret-name>
```

## Common Workflows

### Creating a Secret from a File

```bash
kubectl create secret generic <secret-name> \
  --namespace=<namespace> \
  --from-file=<key>=<path-to-file> \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > <app-name>/templates/sealed-secret.yaml
```

### Creating a Secret with Multiple Keys

```bash
kubectl create secret generic <secret-name> \
  --namespace=<namespace> \
  --from-literal=username='admin' \
  --from-literal=password='super-secret' \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > <app-name>/templates/sealed-secret.yaml
```

### Updating an Existing Secret

1. Create a new sealed secret with the updated value
2. Replace the existing file in `templates/`
3. Commit and push - ArgoCD will sync the changes

```bash
# Generate new sealed secret (overwrites existing file)
kubectl create secret generic <secret-name> \
  --namespace=<namespace> \
  --from-literal=<key>=<new-value> \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > <app-name>/templates/sealed-secret.yaml
```

### Viewing a Decrypted Secret (on cluster)

```bash
# Get the actual secret value from the cluster
kubectl get secret <secret-name> -n <namespace> -o jsonpath='{.data.<key>}' | base64 -d
```

## File Structure

Place sealed secrets in the application's `templates/` directory:

```
<app-name>/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── sealed-secret.yaml          # Your encrypted secret
    └── other-sealed-secret.yaml    # Additional secrets if needed
```

## Important Notes

- ⚠️ **Never commit regular Secrets to git** - only SealedSecrets
- ✅ **SealedSecrets are safe to commit** - they're encrypted
- 🔑 **Keep your cluster's private key safe** - stored as a Kubernetes secret
- 🔄 **Backup the sealing key** if you need to restore the cluster
- 📝 **Document which secrets exist** - add comments in values.yaml

## Backup Sealing Keys

Save these keys securely outside the cluster:

```bash
# Backup the sealing key
kubectl get secret -n kube-system \
  -l sealedsecrets.bitnami.com/sealed-secrets-key=active \
  -o yaml > sealed-secrets-key-backup.yaml

# Store this file securely (password manager, encrypted backup, etc.)
# DO NOT commit this file to git!
```

## Troubleshooting

### SealedSecret not decrypting

```bash
# Check sealed secrets controller logs
kubectl logs -n kube-system -l name=sealed-secrets-controller

# Check if SealedSecret was created
kubectl get sealedsecrets -n <namespace>

# Check if regular Secret was created
kubectl get secrets -n <namespace>
```

### Need to re-seal after cluster restore

If you restore from backup and the sealing key changed:

1. Restore the sealing key from backup (see Backup section)
2. Or re-seal all secrets with the new key

### Getting the public key

```bash
# Fetch the public cert (safe to share)
kubeseal --fetch-cert > public-key.pem
```

## Examples in This Repo

### cribl-edge - Using existingSecret with specific key name

The cribl-edge chart requires the secret key to be named `CRIBL_DIST_LEADER_URL`:

```bash
# Create sealed secret with the specific key name the chart expects
cat <<'EOF' | kubeseal -o yaml > cribl-edge/templates/sealed-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: cribl-edge-leader
  namespace: app-cribl-edge
type: Opaque
stringData:
  CRIBL_DIST_LEADER_URL: "tls://TOKEN@server.cribl.cloud?group=homelab"
EOF
```

Then reference it in `values.yaml`:
```yaml
edge:
  cribl:
    existingSecretForLeader: cribl-edge-leader
```

**Lesson**: Always check the upstream chart's templates to see:
1. What key names it expects in secrets
2. If it supports `existingSecret` or similar parameters

## Additional Resources

- [Sealed Secrets GitHub](https://github.com/bitnami-labs/sealed-secrets)
- [Sealed Secrets Docs](https://sealed-secrets.netlify.app/)
- [kubeseal CLI Reference](https://github.com/bitnami-labs/sealed-secrets#usage)

## Alternative Solutions (Not Used)

For reference, other secret management options include:
- **External Secrets Operator**: Sync from external vaults (AWS, Vault, etc.)
- **SOPS**: File-based encryption with GPG/age
- **ArgoCD Vault Plugin**: Direct Vault integration
- **Git-crypt**: Transparent file encryption in git

Sealed Secrets was chosen for its simplicity and no external dependencies.
