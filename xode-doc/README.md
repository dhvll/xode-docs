# Website

This website is built using [Docusaurus](https://docusaurus.io/), a modern static website generator.

## Installation

```bash
yarn
```

## Local Development

```bash
yarn start
```

This command starts a local development server and opens up a browser window. Most changes are reflected live without having to restart the server.

## Build

```bash
yarn build
```

This command generates static content into the `build` directory and can be served using any static contents hosting service.

## Deployment

### GitHub Pages via GitHub Actions (recommended)

This repository includes a workflow at `.github/workflows/deploy-docs.yml` that builds and deploys the site automatically on pushes to `main`.

1. Ensure `docusaurus.config.ts` has the correct values:
   - `url: 'https://<your-org>.github.io'`
   - `baseUrl: '/<your-repo>/'`
   - `organizationName: '<your-org>'`
   - `projectName: '<your-repo>'`

2. In GitHub → Settings → Pages:
   - Source: GitHub Actions

3. Push to `main`. The workflow will:
   - Install deps, build the site from `xode-doc/`
   - Upload `xode-doc/build` to GitHub Pages

Your site will be available at `https://<your-org>.github.io/<your-repo>/`.

### One-off local deploy (optional)

Using SSH:

```bash
USE_SSH=true yarn deploy
```

Not using SSH:

```bash
GIT_USER=<Your GitHub username> yarn deploy
```

If you are using GitHub pages for hosting, this command builds the website and pushes to the `gh-pages` branch.

### Custom domain

1. In GitHub → Settings → Pages, set your custom domain (e.g., `docs.xode.net`).
2. Add a DNS CNAME record pointing your domain to `<your-org>.github.io`.
3. Optionally create `static/CNAME` with the domain name to persist it across deploys.
