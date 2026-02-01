# Sealed Secret Instructions for kuma-ingress-watcher

Before deploying kuma-ingress-watcher, you need to create a sealed secret containing your Uptime Kuma credentials.

## Prerequisites

- Access to your Kubernetes cluster
- `kubeseal` CLI installed
- Your Uptime Kuma username and password (the credentials you use to log in to the web UI)

## Creating the Sealed Secret

Run the following command, replacing the placeholder values with your actual credentials:

```bash
kubectl create secret generic kuma-ingress-watcher-credentials \
  --namespace=app-uptime-kuma \
  --from-literal=username='YOUR_UPTIME_KUMA_USERNAME' \
  --from-literal=password='YOUR_UPTIME_KUMA_PASSWORD' \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > kuma-ingress-watcher/templates/sealed-secret.yaml
```

## Example Output

The sealed secret should look something like this:

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: kuma-ingress-watcher-credentials
  namespace: app-uptime-kuma
spec:
  encryptedData:
    username: AgBsuoV02XVdiTy07xFv... (encrypted data)
    password: AgCKf8x9z3HdPqR5... (encrypted data)
  template:
    metadata:
      name: kuma-ingress-watcher-credentials
      namespace: app-uptime-kuma
    type: Opaque
```

## Important Notes

- **Never commit plaintext secrets** - only the sealed (encrypted) version
- The sealed secret can only be decrypted by your cluster's Sealed Secrets controller
- If you rotate credentials in Uptime Kuma, you'll need to create a new sealed secret

## After Creating the Secret

1. Verify the sealed secret was created: `ls kuma-ingress-watcher/templates/sealed-secret.yaml`
2. Commit and push the changes to both repos
3. ArgoCD will automatically sync and deploy the application
