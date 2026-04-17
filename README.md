# gha-workflows

Shared reusable GitHub Actions workflows for personal projects.

## Workflows

| Workflow | Purpose | Used by |
|----------|---------|---------|
| `tag-release-changelog.yml` | Create GitHub Release with auto-generated changelog | synqevent, synqbeam, synqevent-ios, gha-workflows |
| `release.yml` | Self-release trigger: on `v*` tag push, calls `tag-release-changelog.yml` locally | gha-workflows (this repo) |
| `nextjs-lint.yml` | ESLint + TypeScript type checking for Next.js | synqevent, synqbeam |
| `ios-build-test.yml` | Build & test iOS project on Simulator | synqevent-ios |
| `ios-appstore.yml` | Archive & upload IPA to App Store Connect | synqevent-ios |

## Versioning this repo

This repo versions itself. Push a semver tag and `release.yml` fires, which calls `tag-release-changelog.yml` to create the GitHub Release with auto-generated notes:

```bash
git tag v1.0.0
git push origin v1.0.0
```

Consumers pin to `@main` today; once stable, pin to a tag (e.g. `@v1.0.0`) for reproducibility.

## Usage

### Tag Release & Changelog

```yaml
jobs:
  changelog:
    uses: perezmark/gha-workflows/.github/workflows/tag-release-changelog.yml@main
    with:
      tag_name: ${{ github.ref_name }}
    permissions:
      contents: write
```

### Next.js Lint & Typecheck

```yaml
jobs:
  lint:
    uses: perezmark/gha-workflows/.github/workflows/nextjs-lint.yml@main
    with:
      node_version: '20'
      prisma_generate: false
      typecheck: true
```

### iOS Build & Test

```yaml
jobs:
  test:
    uses: perezmark/gha-workflows/.github/workflows/ios-build-test.yml@main
    with:
      scheme: MyApp
      project_path: MyApp.xcodeproj
```

### iOS App Store Connect

```yaml
jobs:
  appstore:
    uses: perezmark/gha-workflows/.github/workflows/ios-appstore.yml@main
    with:
      scheme: MyApp
      project_path: MyApp.xcodeproj
      bundle_id: com.example.app
      team_id: 'TEAMID'
      provisioning_profile_name: 'My Distribution Profile'
    secrets:
      DISTRIBUTION_CERTIFICATE_P12_BASE64: ${{ secrets.DISTRIBUTION_CERTIFICATE_P12_BASE64 }}
      DISTRIBUTION_CERTIFICATE_PASSWORD: ${{ secrets.DISTRIBUTION_CERTIFICATE_PASSWORD }}
      PROVISIONING_PROFILE_BASE64: ${{ secrets.PROVISIONING_PROFILE_BASE64 }}
      APP_STORE_CONNECT_KEY_ID: ${{ secrets.APP_STORE_CONNECT_KEY_ID }}
      APP_STORE_CONNECT_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_ISSUER_ID }}
      APP_STORE_CONNECT_PRIVATE_KEY_BASE64: ${{ secrets.APP_STORE_CONNECT_PRIVATE_KEY_BASE64 }}
```
