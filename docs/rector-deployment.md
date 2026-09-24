# Rector deployment

This fork deploys the pinned Cloudflare OS release to `os.rector.co.jp` in the
`daichi.hiroki` Cloudflare account. The upstream source remains a Git submodule;
Rector-specific settings live in `deployment.jsonc`.

## Identity and access

Cloudflare Access protects the router hostname. Use a self-hosted Access
application for exactly `os.rector.co.jp` and a Generic OIDC identity provider
named `Rector Auth`:

| Setting | Value |
| --- | --- |
| Client ID | `cloudflare-os` |
| Authorization URL | `https://auth.rector.co.jp/oidc/authorize` |
| Token URL | `https://auth.rector.co.jp/oidc/token` |
| Certificate URL | `https://auth.rector.co.jp/.well-known/jwks.json` |
| Scopes | `openid email profile` |
| PKCE | S256 enabled |
| Callback | `https://still-king-8376.cloudflareaccess.com/cdn-cgi/access/callback` |

The client secret is kept outside this repository in Rector's secret store as
`RECTOR_CLOUDFLARE_OS_OIDC_CLIENT_SECRET`. Rector Auth receives only its hash in
the Worker secret `OIDC_ACCESS_CLIENT_SECRET_HASH`. The Access application should
offer only the `Rector Auth` login method. Its allow policy should select that
identity provider; Rector Auth decides which email addresses may sign in.
`daichi.hiroki@rector.co.jp` is the sole Cloudflare OS administrator.
Use a 30-minute Access application session so changes to the Rector Auth
allowlist are enforced on the next login.

The existing Access team name and callback must match the URL configured in
`officework/auth/wrangler.jsonc`. Do not rename this team: it already protects
another application. The Access application ID is
`3444f190-e046-4721-8ab0-4b207e3ddfc0`; its audience tag is pinned in
`deployment.jsonc`. Keep the application and audience stable for this deployment.

## Deployment order

1. Complete the Rector Auth migration, Worker secret, and deployment. Verify the
   discovery document and JWKS endpoint.
2. Create or confirm the Access team, Generic OIDC identity provider, Access
   application, and policy. Copy its audience into `deployment.jsonc`.
3. Run `pnpm check`, then `pnpm deploy` with Node 24.19+ and pnpm 11.17+.
4. Test a signed-out visit, an allowed administrator, an allowed non-admin, and
   a denied identity. Confirm that the five private Workers have no public
   routes and that only the router serves `os.rector.co.jp`. Leave the example
   Custom Gatekeeper disabled in `/admin` until its behavior is reviewed.

The first deployment provisioned three KV namespaces and one R2 bucket. Their
identities are pinned in `deployment.jsonc`: Context KV
`cb898b090f6547f4959310cec34805f1`, Blueprints KV
`3e6bab2870cb4f0fa885bc83e94e3b71`, Avatars KV
`2583dce8d7ab4c768a3ad362111bdd33`, and R2 bucket
`rector-os-workshop-blueprint-content`. Keep these IDs and the six Worker names
stable to retain data.
