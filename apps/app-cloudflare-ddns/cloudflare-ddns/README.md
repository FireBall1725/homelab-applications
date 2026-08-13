# cloudflare-ddns

Keeps the public `home.firekatt.ca` A record pointed at the site's current WAN IP. The ISP hands out a dynamic residential IP that rotates roughly daily, so this checks the real public IP every 5 minutes & updates Cloudflare when it changes.

Runs [`favonia/cloudflare-ddns`](https://github.com/favonia/cloudflare-ddns) as a single Deployment in `app-cloudflare-ddns`. It detects the WAN IP from inside the pod by querying Cloudflare's `cloudflare.trace` endpoint, so it does not need host networking or a port-forward.

## Config

Set through env vars in `deployment.yaml`:

- `DOMAINS` is `home.firekatt.ca`.
- `PROXIED` is `false`. The record holds the real IP for remote access, so it stays DNS-only (gray cloud); Cloudflare's proxy can't pass arbitrary ports.
- `IP6_PROVIDER` is `none`. No IPv6 at this site, so AAAA management is off.
- `UPDATE_CRON` is `@every 5m`.

## Token

`sealed-secret.yaml` holds `CLOUDFLARE_API_TOKEN`, re-sealed from the same Cloudflare token cert-manager uses for DNS-01. That token already carries Zone:Read & DNS:Edit on `firekatt.ca`, which is the exact scope favonia needs to read the zone & write the A record. Rotating the token means re-sealing it in both `cert-manager` & `app-cloudflare-ddns`.
