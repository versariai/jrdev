# jrdev.ai site

The Phase 0 homepage from [plan/phase0/homepage.md](../plan/phase0/homepage.md): one page plus a privacy page, static Astro output.

```sh
npm install
npm run dev     # http://localhost:4321
npm run build   # static files in dist/
```

Before launch, set the values in `src/config.ts`, replace every `TBD` on the privacy page, and run the plan's launch checklist. Changing any consent-relevant copy means bumping `CONSENT_VERSION` and adding `consent/vN.md`.

Deploy: Cloudflare Pages, root directory `site`, build command `npm run build`, output `dist`.
