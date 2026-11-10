# fireguieditor

Chart for the FireGui editor: web UI, REST API and MCP server (`/mcp`) in one pod, with the
Java render worker as a second process in the same container.

- **Image:** `ghcr.io/fireball1725/fireguieditor`. The release workflow pins the tag in
  `values.yaml` through a PR, because ArgoCD watches git, not the registry.
- **Database:** a CloudNativePG cluster, `fireguieditor-db`. The app migrates on start.
- **Secret:** `templates/sealed-secret-api-token.yaml` holds `FIREGUIEDITOR_API_TOKEN`
  (at least 24 characters). To rotate it, seal a new value:

  ```bash
  kubectl create secret generic fireguieditor-api --namespace=app-fireguieditor \
    --from-literal=FIREGUIEDITOR_API_TOKEN="$NEW_TOKEN" --dry-run=client -o yaml | kubeseal -o yaml
  ```

- **Storage:** `/data` (2Gi) caches the Minecraft client jar the worker downloads on first
  use.
- **URL:** `http://fireguieditor.k8s.firekatt.ca`, LAN only.
