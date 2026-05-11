# CLI Release Guide

## Overview

The CLI release process is automated via GitHub Actions. Pushing a `cli-v*` tag triggers the workflow to build, test, and publish the CLI package.

## Prerequisites

### Required Secrets

Configure these in GitHub repository settings → Secrets and variables → Actions:

- `NPM_TOKEN`: npm authentication token with publish permissions
  - Generate at https://www.npmjs.com/settings/YOUR_USERNAME/tokens
  - Use **Classic Automation Token** (bypasses 2FA automatically), or
  - Use **Granular Access Token** with "Allow bypass 2FA" enabled and scoped to the package

### Optional Variables

Configure these in GitHub repository settings → Secrets and variables → Actions → Variables:

- `NPM_REGISTRY`: npm registry URL (default: `https://registry.npmjs.org`)
  - For custom registries (e.g., Verdaccio, GitHub Packages)

### Package Configuration

Ensure `cli/package.json` has the correct organization scope:

```json
{
  "name": "@platypup/skillhub",
  "publishConfig": {
    "access": "public"
  }
}
```

## Release Process

### Tag is the source of truth

The CLI version is derived from the git tag name (`cli-vX.Y.Z`). You don't need to bump `cli/package.json` locally — the workflow does it automatically during the build.

### Create and Push Tag

```bash
# From any clean working tree
git tag cli-v0.1.8
git push origin cli-v0.1.8
```

That's the entire release procedure. The workflow takes over from here.

> `cli/package.json` on `main` may lag behind the latest published version. This is intentional — the tag is the release record, not `package.json`. If you want to keep them in sync, bump `package.json` in a regular PR before tagging.

### Workflow Execution

The workflow performs these steps:

1. **Build and Test**
   - Extract version from tag name (e.g., `cli-v0.1.8` → `0.1.8`)
   - Write version into `cli/package.json`
   - Install dependencies with Bun
   - Run linter, type checker, and tests
   - Build the CLI
   - Verify built version matches the tag

2. **Publish to npm** (if `NPM_TOKEN` is configured)
   - Check if version already exists on registry
   - Publish package if version is new
   - Skip if version already published

3. **Create GitHub Release**
   - Package dist/ + README + LICENSE as tar.gz and zip
   - Generate SHA256 checksums
   - Create GitHub release with artifacts

### Verify Release

Check the following:

- GitHub Actions workflow completed successfully
- GitHub Release created at https://github.com/iflytek/skillhub/releases
- Package published to npm (if configured): `npm view @platypup/skillhub@0.1.8`

## Manual Trigger

You can manually trigger the workflow from GitHub Actions UI:

1. Go to Actions → Release CLI
2. Click "Run workflow"
3. Enter tag name (e.g., `cli-v0.1.8`) — must follow `cli-vX.Y.Z` format
4. Optionally skip npm publish

## Local Testing

Test the publish flow locally without actually publishing:

```bash
# Set up test environment
cd cli
cp .env.example .env.local

# Edit .env.local with test credentials
# NPM_TOKEN=your-test-token
# NPM_ORG=your-test-org
# NPM_REGISTRY=https://registry.npmjs.org

# Dry run
make publish-cli-dry
```

## Troubleshooting

### Version Already Exists

If the workflow fails with "version already exists":

1. Check current published version: `npm view @platypup/skillhub version`
2. Create a new tag with a higher version (e.g., `cli-v0.1.9`)

### npm Publish Fails

- **403 Forbidden with 2FA message**: Token is not an Automation Token or doesn't have bypass 2FA enabled — regenerate with the correct type
- Verify `NPM_TOKEN` secret is valid and has publish permissions
- Check package name matches your npm organization scope
- Ensure `publishConfig.access` is set to `public`

### Build Fails

- Check Bun version matches `packageManager` in package.json
- Verify all tests pass locally: `cd cli && bun test`
- Check linter: `cd cli && bun run lint`

## Tag Naming Convention

- CLI releases: `cli-v*` (e.g., `cli-v0.1.5`)
- Repository releases: `v*` (e.g., `v0.3.0`)

This separation allows independent versioning of CLI and server components.
