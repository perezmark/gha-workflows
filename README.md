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

## Versioning

Consumers should pin to a released version, not `@main`.

| Pin | Behavior |
|-----|----------|
| `@v1` | Floating major — gets non-breaking updates (**recommended**) |
| `@v1.1.0` | Exact version — fully reproducible |
| `@main` | Unstable — picks up every push, including breaking changes (avoid) |

### Releasing a new version

This repo versions itself. Push a semver tag and `release.yml` fires, which calls `tag-release-changelog.yml` to create the GitHub Release with auto-generated notes:

```bash
git tag v1.1.0
git push origin v1.1.0
```

Then update the floating major (`v1`) to point at the new release so consumers pinned to `@v1` pick it up:

```bash
git tag -f v1 v1.1.0
git push origin v1 --force
```

Bump the major (`v2`) when breaking changes land; leave `v1` pointing at the last compatible release.

## Usage

### Tag Release & Changelog

```yaml
jobs:
  changelog:
    uses: perezmark/gha-workflows/.github/workflows/tag-release-changelog.yml@v1
    with:
      tag_name: ${{ github.ref_name }}
    permissions:
      contents: write
```

### Next.js Lint & Typecheck

```yaml
jobs:
  lint:
    uses: perezmark/gha-workflows/.github/workflows/nextjs-lint.yml@v1
    with:
      node_version: '20'
      prisma_generate: false
      typecheck: true
```

### iOS Build & Test

```yaml
jobs:
  test:
    uses: perezmark/gha-workflows/.github/workflows/ios-build-test.yml@v1
    with:
      scheme: MyApp
      project_path: MyApp.xcodeproj
```

### iOS App Store Connect

```yaml
jobs:
  appstore:
    uses: perezmark/gha-workflows/.github/workflows/ios-appstore.yml@v1
    with:
      scheme: MyApp
      project_path: MyApp.xcodeproj
      bundle_id: com.example.app
      team_id: 'TEAMID'
      provisioning_profile_name: 'My Distribution Profile'
      # xcode_version defaults to '26.3' — override if needed
    secrets:
      DISTRIBUTION_CERTIFICATE_P12_BASE64: ${{ secrets.DISTRIBUTION_CERTIFICATE_P12_BASE64 }}
      DISTRIBUTION_CERTIFICATE_PASSWORD: ${{ secrets.DISTRIBUTION_CERTIFICATE_PASSWORD }}
      PROVISIONING_PROFILE_BASE64: ${{ secrets.PROVISIONING_PROFILE_BASE64 }}
      APP_STORE_CONNECT_KEY_ID: ${{ secrets.APP_STORE_CONNECT_KEY_ID }}
      APP_STORE_CONNECT_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_ISSUER_ID }}
      APP_STORE_CONNECT_PRIVATE_KEY_BASE64: ${{ secrets.APP_STORE_CONNECT_PRIVATE_KEY_BASE64 }}
```

#### Required project configuration

The consumer Xcode project must have **manual signing** configured on the target's **Release** build config (not via CLI overrides — SPM dependencies reject global signing settings). In `.xcodeproj/project.pbxproj` under the Release config:

```
CODE_SIGN_STYLE = Manual;
CODE_SIGN_IDENTITY = "Apple Distribution";
PROVISIONING_PROFILE_SPECIFIER = "My Distribution Profile";
DEVELOPMENT_TEAM = TEAMID;
```

Debug config can stay on `Automatic` for local development.
