# kisuke

Kisuke is a Discord bot for the mod server. It answers FireBall's mentions and DMs through
Claude, files tickets in `FireBall1725/mod-issues` after an approving reply, and helps people
FireBall sends it to. Source: <https://github.com/fireball1725/firediscordmodbot>.

## Deploying a version

The image is `ghcr.io/fireball1725/firediscordmodbot`, private, pulled with the `ghcr-pull`
secret. Pin the immutable tag in `values.yaml` and merge it; ArgoCD watches git, not the
registry. rc and stable releases pin themselves through the release workflow
(`deploy-app: kisuke`). Nightly images are pinned by hand.

## Secrets

Both are sealed in `templates/` and scoped to the `app-kisuke` namespace.

- `kisuke-secrets`: `DISCORD_TOKEN`, `ANTHROPIC_API_KEY`, `GITHUB_APP_PRIVATE_KEY` (base64 of the
  Urahara-Shop app key), `ALLOWED_USER_IDS`, and `KISUKE_HINDSIGHT_AUTH` (optional). The app and
  installation ids are not secret and sit in the ConfigMap.
- `ghcr-pull`: a pull credential for the private image.

## Notes

- One replica with Recreate. A second gateway connection would answer every message twice.
- `/api/v1/ready` fails while the Discord gateway is down, so a disconnected bot leaves the Service.
- The database is the `kisuke-db` CloudNativePG cluster; `DATABASE_URL` comes from its `kisuke-db-app` secret.
