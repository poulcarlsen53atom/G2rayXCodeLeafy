# G2ray Codespace Waker

This Cloudflare Worker gives you a private manual `/wake` endpoint that starts one GitHub Codespace through GitHub's official Codespaces API.

It does not keep the Codespace alive forever and it cannot bypass quota, billing, deletion, or account restrictions.

## 1. Create a GitHub Token

Use one of these:

- Classic personal access token with the `codespace` scope.
- Fine-grained token that can access the repo and has Codespaces lifecycle/admin write permission, if that option is available in your account.

Keep this token private. Do not commit it to git.

## 2. Configure Wrangler

Install or use Wrangler:

```bash
npm create cloudflare@latest -- --help
npx wrangler --version
```

Copy the example config:

```bash
cd worker/codespace-waker
cp wrangler.toml.example wrangler.toml
```

Edit `wrangler.toml` and set:

```toml
CODESPACE_NAME = "your-codespace-name"
```

Example:

```toml
CODESPACE_NAME = "animated-spork-wvr97qjxqjqwcg6xq"
```

## 3. Add Secrets

Create a long random wake secret. Examples:

```bash
openssl rand -hex 32
```

PowerShell:

```powershell
[guid]::NewGuid().Guid + [guid]::NewGuid().Guid
```

Store secrets in Cloudflare:

```bash
npx wrangler secret put GITHUB_TOKEN
npx wrangler secret put WAKE_SECRET
```

Paste the GitHub token for `GITHUB_TOKEN`.
Paste your random wake secret for `WAKE_SECRET`.

## 4. Deploy

```bash
npx wrangler deploy
```

Wrangler prints the Worker URL, for example:

```text
https://g2ray-codespace-waker.YOUR_SUBDOMAIN.workers.dev
```

## 5. Start The Codespace Manually

Recommended CLI call:

```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_WAKE_SECRET" \
  https://g2ray-codespace-waker.YOUR_SUBDOMAIN.workers.dev/wake
```

You can also open the Worker URL in a browser and enter the wake secret in the form.

## Expected Responses

- `200` with `ok: true`: GitHub accepted or handled the start request.
- `401`: Wrong wake secret.
- `402`: GitHub quota or billing blocked the start. Wait for quota reset or adjust GitHub billing settings.
- `403`: GitHub token is rejected or missing the right scope.
- `404`: Codespace name is wrong or the token cannot access it.

## Security Notes

- Do not put `GITHUB_TOKEN` or `WAKE_SECRET` in `wrangler.toml`.
- Do not commit `.dev.vars`.
- Prefer the `Authorization: Bearer ...` header over putting secrets in URLs.
- Rotate the GitHub token if the Worker or Cloudflare account is compromised.
