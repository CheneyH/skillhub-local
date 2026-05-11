# CLI Release Guide

## Overview

The CLI release process is automated via GitHub Actions. Pushing a `cli-v*` tag triggers the workflow to build, test, and publish the CLI package.

## Prerequisites

### Required Secrets

Configure these in GitHub repository settings → Secrets and variables → Actions:

- `NPM_TOKEN`: npm authentication token with publish permissions
  - Generate at https://www.npmjs.com/settings/YOUR_USERNAME/tokens
  - Select "Automation" token type

### Optional Variables

Configure these in GitHub repository settings → Secrets and variables → Actions → Variables:

- `NPM_REGISTRY`: npm registry URL (default: `https://registry.npmjs.org`)
  - For custom registries (e.g., Verdaccio, GitHub Packages)

### Package Configuration

Ensure `cli/package.json` has the correct organization scope:

```json
{
  "name": "@your-org/skillhub",
  "publishConfig": {
    "access": "public"
  }
}
```

## Release Process

### 1. Update Version

Update the version in `cli/package.json`:

```bash
cd cli
npm version patch  # or minor, or major
```

This updates `package.json` and creates a commit.

### 2. Create and Push Tag

```bash
# Create tag matching the new version
git tag cli-v0.1.5

# Push tag to trigger workflow
git push origin cli-v0.1.5
```

### 3. Workflow Execution

The workflow performs these steps:

1. **Build and Test**
   - Install dependencies with Bun
   - Run linter, type checker, and tests
   - Build the CLI
   - Verify built version matches package.json

2. **Publish to npm** (if `NPM_TOKEN` is configured)
   - Check if version already exists on registry
   - Publish package if version is new
   - Skip if version already published

3. **Create GitHub Release**
   - Package dist/ + README + LICENSE as tar.gz and zip
   - Generate SHA256 checksums
   - Create GitHub release with artifacts

### 4. Verify Release

Check the following:

- GitHub Actions workflow completed successfully
- GitHub Release created at https://github.com/iflytek/skillhub/releases
- Package published to npm (if configured): `npm view @your-org/skillhub@0.1.5`

## Manual Trigger

You can manually trigger the workflow from GitHub Actions UI:

1. Go to Actions → Release CLI
2. Click "Run workflow"
3. Enter tag name (e.g., `cli-v0.1.5`)
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

1. Check current published version: `npm view @your-org/skillhub version`
2. Update `cli/package.json` to a newer version
3. Create a new tag

### npm Publish Fails

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
