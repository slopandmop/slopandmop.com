# slopandmop.com

Static landing for [slopandmop][sm] — a one-button front door to the sm factory.

## Shape

Pure static HTML on GitHub Pages. Everything dynamic lives behind
`factory.slopandmop.com` (a dd-CP-shaped axum service on its own TDX VM):

```
Visitor → slopandmop.com/
          click "Start your sm"
          → POST factory.slopandmop.com/sm/provision
          ← { chat_url, ttl_seconds }
          window.location = chat_url
```

The factory:

1. Bakes an `ee-config` (sm_id, cookie signing key, boot workloads, Anthropic API key).
2. Runs `gcloud compute instances create --confidential-compute-type=TDX
   --image-family=slopandmop-stable --metadata-from-file=ee-config=...`.
3. Polls `sm.sm-<id>.slopandmop.com/health` until ready.
4. Mints a signed session cookie and returns a redirect URL to
   `chat.sm-<id>.slopandmop.com/?session=<cookie>`.

No credentials live in this repo or in the browser. The factory holds
`compute.admin` for our GCP project and the Anthropic API key; this page
just invites the click.

## Layout

```
.
├── CNAME                      # slopandmop.com
├── public/
│   ├── index.html             # hero + one button
│   └── terms.html
└── .github/workflows/pages.yml
```

## Local preview

```
python3 -m http.server 8080 --directory public
open http://localhost:8080
```

The button will try to fetch `https://factory.slopandmop.com/sm/provision` —
CORS-gated against `slopandmop.com`, so localhost requests get rejected. For
full-stack development, point `FACTORY_URL` at a local factory (see
[slopandmop][sm] `src/factory.rs`) and relax CORS.

## Deploy

Push to `main`. GitHub Pages rebuilds from `public/` via `.github/workflows/pages.yml`.

[sm]: https://github.com/slopandmop/slopandmop
