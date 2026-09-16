# Dharmendra Labs

Static pages deployed to Cloudflare Workers using Static Assets. No Worker script or backend is used.

The existing top-level app folders are the current GitHub Pages sources used by published App Store apps. Leave them unchanged until every app has been migrated. Cloudflare deploys only the separate `cloudflare/public/` copies.

## Local development

From the repository root:

```sh
npx wrangler dev
```

Wrangler prints the local URL. Open `/test` to verify clean-path routing.

## Deploy through GitHub

Cloudflare Workers Builds deploys pushes to the production branch configured in the dashboard. GitHub's default branch and the existing GitHub Pages source are `main`; confirm that Cloudflare also uses `main` for production.

Configure Workers Builds with these settings:

- Root directory: the repository root (`/`).
- Build command: leave blank (these are static files).
- Deploy command: `npx wrangler deploy`.
- Non-production branch deploy command: `npx wrangler versions upload`, if preview builds are enabled.

Commit and push a feature branch, check its Cloudflare preview when enabled, then merge into the configured production branch. Cloudflare performs authentication and deployment; local `wrangler login` and manual deployment are optional.

For an optional manual deployment from the repository root:

```sh
npx wrangler deploy
```

After the first deployment, add `labs.dharmendra.tech` as the Worker's custom domain in the Cloudflare dashboard. The domain is intentionally not configured in `wrangler.jsonc`.

Cloudflare can access a private GitHub repository when its GitHub app has permission. However, on GitHub Free, changing this repository to private unpublishes its existing GitHub Pages site. Keep those App Store URLs available throughout migration, including for installed app versions still using them.

References: [Cloudflare build configuration](https://developers.cloudflare.com/workers/ci-cd/builds/configuration/), [GitHub visibility changes](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility).

## Hosted app pages

The Cloudflare asset directory is organized by app:

```text
cloudflare/public/
  littleplot/
    index.html
    privacy.html
  nirogyam/
    index.html
    privacy.html
    privacy-v2.0.html
    privacy-v2.1.html
    support-v2.1.html
    privacy-v2.2.html
    support-v2.2.html
  rishibharat/
    index.html
    privacy.html
  veerbharat/
    index.html
    privacy.html
```

Each app's `index.html` is served at its app path, for example `/littleplot` or `/nirogyam`. Other HTML files also use extensionless paths, such as `/littleplot/privacy` and `/nirogyam/privacy-v2.1`.

## Add another app in the future

Create `cloudflare/public/<app-name>/index.html` for its main support page and, when needed, `cloudflare/public/<app-name>/privacy.html` for its privacy policy. Keep that app's CSS, JavaScript, and images inside the same folder. No Wrangler configuration change is needed.

Use root-relative links with the app prefix, for example `/littleplot/style.css`, `/littleplot/icon.png`, and `/littleplot/privacy`. Folder index pages are served without a trailing slash, so a relative link such as `style.css` on `/littleplot` would incorrectly resolve to `/style.css`.

Every response receives an `X-Robots-Tag: noindex, nofollow` header from `cloudflare/public/_headers`. HTML pages should also retain this metadata in `<head>`:

```html
<meta name="robots" content="noindex, nofollow">
```

Do not place secrets or private API keys anywhere under `cloudflare/public/`; everything in that directory is publicly downloadable after deployment.
