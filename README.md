# Reusable CI/CD Workflows

This repository provides reusable GitHub Actions workflows and composite actions for CI, deployments, and library publishing.

## Available Workflows

### `.github/workflows/simple-ci.yml`

Reusable CI pipeline with build, check, and test steps.

Inputs:

- `package-manager` (optional, default: `npm`) - `npm`, `pnpm`, or `bun`
- `node-version` (optional, default: `24`)

### `.github/workflows/staging-deploy.yml`

Builds and pushes staging images for each app, then triggers Dokploy deployments.

Inputs:

- `apps` (required) - JSON array of app names
- `dokploy-app-ids` (required) - JSON object mapping app name to Dokploy app ID
- `image-name-prefix` (optional, default: empty)

Secrets:

- `dockerhub-username`
- `dockerhub-token`
- `dokploy-domain`
- `dokploy-api-key`

### `.github/workflows/prod-deploy.yml`

Deploys production from a release tag using app/version parsing.

Expected tag format:

- `<app-name>@<semver>`
- Examples: `backend@1.2.3`, `frontend@2.0.0-rc.1`

Inputs:

- `apps` (required) - JSON array of deployable app names
- `dokploy-app-ids` (required) - JSON object mapping app name to Dokploy app ID
- `image-name-prefix` (optional, default: empty)

Secrets:

- `dockerhub-username`
- `dockerhub-token`
- `dokploy-domain`
- `dokploy-api-key`

### `.github/workflows/publish-library.yml`

Publishes a package based on the pushed tag.

Expected tag format:

- `<library-name>@<semver>`
- Examples: `core@1.0.0`, `ui@1.5.0-alpha.2`

Inputs:

- `package-manager` (optional, default: `npm`) - `npm`, `pnpm`, or `bun`
- `node-version` (optional, default: `24`)
- `libraries` (optional, default: `[]`) - JSON array of allowed library names

Secrets:

- `auth-token` (npm auth token)

Publish tag behavior:

- Stable versions publish with `latest`
- Prerelease versions publish with the prerelease identifier
- Examples:
  - `1.2.3` -> `latest`
  - `1.2.3-alpha.1` -> `alpha`
  - `1.2.3-beta.2` -> `beta`
  - `1.2.3-next.4` -> `next`

## Composite Actions

### `.github/actions/tag-extraction`

Parses tag information and validates the app/library name against a JSON allowlist.

Input:

- `apps` (required) - JSON array of accepted names

Outputs:

- `app-name`
- `version`
- `is-prerelease`
- `release-tag` (prerelease channel or `latest`)

### `.github/actions/publish-package`

Publishes with:

- `npm publish --tag <tag>`
- `pnpm publish --tag <tag>`
- `bun publish --tag <tag>`

## Example Usage

### Call CI

```yaml
name: CI

on:
	pull_request:

jobs:
	ci:
		uses: melv1c/github-workflows/.github/workflows/simple-ci.yml@main
		with:
			package-manager: pnpm
			node-version: '24'
```

### Call Production Deploy

```yaml
name: Production Deploy

on:
	push:
		tags:
			- '*@*'

jobs:
	deploy:
		uses: melv1c/github-workflows/.github/workflows/prod-deploy.yml@main
		with:
			apps: '["backend","frontend"]'
			dokploy-app-ids: '{"backend":"app-id-1","frontend":"app-id-2"}'
			image-name-prefix: myorg-
		secrets:
			dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}
			dockerhub-token: ${{ secrets.DOCKERHUB_TOKEN }}
			dokploy-domain: ${{ secrets.DOKPLOY_DOMAIN }}
			dokploy-api-key: ${{ secrets.DOKPLOY_API_KEY }}
```

### Call Library Publish

```yaml
name: Publish Library

on:
	push:
		tags:
			- '*@*'

jobs:
	publish:
		uses: melv1c/github-workflows/.github/workflows/publish-library.yml@main
		with:
			package-manager: pnpm
			libraries: '["core","ui","utils"]'
		secrets:
			auth-token: ${{ secrets.NPM_TOKEN }}
```
