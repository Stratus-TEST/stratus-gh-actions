# Simple Version Bump and Release Action

A lightweight composite GitHub Action (Bash + GitHub Script) that automates version bumping, tag creation, and release generation using GitHub's native release notes. No Docker or Node.js runtime required.

## Features

- 🔄 Automatic version bumping based on PR labels or commit messages
- 🏷️ Git tag creation and management (including major version and "latest" tags) **via the GitHub API for Verified status**
- 📝 Native GitHub release notes generation (no external dependencies)
- 🎯 Simple and reliable
- 📋 Zero configuration required
- ✅ Works for both PR merges and direct pushes to main

## How It Works

1. **For Pull Requests**: Reads PR labels to determine version bump
2. **For Push Events**: Parses the latest commit message on the branch for version keywords
3. Creates a new semantic version tag (e.g., v1.2.3)
4. Updates the major version tag (e.g., v1) to reference the latest release in that major version
5. Updates the "latest" tag to always point to the most recent release
6. Generates a GitHub release with native release notes

## Version Bumping Rules

### PR Labels (Priority)

- `breaking-change` or `major` → Major version (vX.0.0)
- `enhancement`, `feature`, or `minor` → Minor version (v0.X.0)
- Any other labels or no labels → Patch version (v0.0.X)

### Commit Messages (Fallback)

- `breaking change:` or `major:` → Major version
- `feat:` or `minor:` → Minor version
- `fix:` or `patch:` → Patch version
- Default → Patch version

## Usage

### Basic Usage

```yaml
name: Release

on:
  push:
    branches: [main]
  pull_request:
    types: [closed]
    branches: [main]

permissions:
  contents: write
  pull-requests: write

jobs:
  release:
    if: github.event_name == 'push' || github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Create Release
        uses: HafslundEcoVannkraft/stratus-actions/release@v3
```

### With Options

```yaml
- name: Create Release
  uses: HafslundEcoVannkraft/stratus-actions/release@v3
  with:
    draft: false # Create as published release (default: false)
    prerelease: true # Mark as pre-release
```

### With Tag Prefix (for test tags)

```yaml
- name: Create Test Release
  uses: HafslundEcoVannkraft/stratus-actions/release@v3
  with:
    tag-prefix: test-
    draft: true
    prerelease: true
```

## Inputs

| Input        | Description                                                | Required | Default |
| ------------ | ---------------------------------------------------------- | -------- | ------- |
| `draft`      | Create release as draft                                    | No       | `false` |
| `prerelease` | Mark as pre-release                                        | No       | `false` |
| `tag-prefix` | Prefix for all tags (e.g., `test-`). Useful for test runs. | No       | `""`    |

## Outputs

| Output             | Description                | Example                                             |
| ------------------ | -------------------------- | --------------------------------------------------- |
| `new_version`      | Generated semantic version | `v1.2.3`                                            |
| `previous_version` | Previous version           | `v1.2.2`                                            |
| `bump_type`        | Version increment type     | `major`, `minor`, or `patch`                        |
| `release_url`      | URL of created release     | `https://github.com/owner/repo/releases/tag/v1.2.3` |

## Examples

### Release on Merge to Main

```yaml
name: Release on Merge

on:
  pull_request:
    types: [closed]
    branches: [main]

permissions:
  contents: write
  pull-requests: write

jobs:
  release:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Create Release
        uses: HafslundEcoVannkraft/stratus-actions/release@v3
```

### Release on Direct Push

```yaml
name: Release on Push

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Create Release
        uses: HafslundEcoVannkraft/stratus-actions/release@v3
```

### Draft Release for Review

```yaml
- name: Create Draft Release
  uses: HafslundEcoVannkraft/stratus-actions/release@v3
  with:
    draft: true
```

## Release Notes

This action uses GitHub's native release notes generation, which:

- Automatically categorizes PRs based on labels
- Lists contributors
- Provides a full changelog
- Can be configured via `.github/release.yml`

To customize release notes categories, create `.github/release.yml`:

```yaml
changelog:
  categories:
    - title: 🚀 Features
      labels:
        - enhancement
        - feature
    - title: 🐛 Bug Fixes
      labels:
        - bug
        - bugfix
```

## Migration from v1

The v2 release removes all AI-powered features and external dependencies:

**Removed:**

- Azure OpenAI integration
- AI-generated release notes
- Complex configuration options
- External API dependencies

**What stays the same:**

- Version bumping logic
- Tag creation
- Basic release creation

**What's new:**

- Uses GitHub's native release notes
- Zero configuration required
- More reliable and faster
- No API costs

## Why Use This Action?

- **Simple**: No complex configuration or external dependencies
- **Reliable**: Uses only GitHub's native features
- **Fast**: No external API calls
- **Free**: No costs for AI or external services
- **Maintainable**: Minimal code, easy to understand

## Tag Management

This action manages three levels of Git tags for your releases, **always using the GitHub API (never git) so all tags are Verified**:

1. **Specific version tags** (e.g., `v1.2.3` or `test-v1.2.3`): Created for each release based on version bumping rules and the prefix.
2. **Major version tags** (e.g., `v1` or `test-v1`): Automatically updated to point to the latest release within that major version and prefix.
3. **Latest tag** (`latest` or `test-latest`): Always points to the most recent release for the given prefix.

### Tag Usage Benefits

- **Versioned dependencies**: Users can pin to specific versions (`v1.2.3`).
- **Major version stability**: Users can reference major versions (`v1`) to get the latest updates without breaking changes.
- **Latest releases**: Users can reference `latest` to always use the most current version.

All tags are created and updated using the GitHub API, never with git, so they always show as **Verified** in the GitHub UI.

These tags are force-updated with each release to ensure they always point to the correct commit.

## Troubleshooting & FAQ

- **Q: Are the tags/releases created by this action "Verified"?**
  - A: Yes, all tags and releases are created and updated via the GitHub API (never git), so they are always signed by GitHub and show as "Verified" in the commit history.
- **Q: Can I use this action in a fork or mirror repo?**
  - A: Yes, but ensure you have write permissions and the correct token setup for your use case.
- **Q: How do I create test tags that are easy to clean up?**
  - A: Use the `tag-prefix` input (e.g., `tag-prefix: test-`) to create all tags with a `test-` prefix. This makes it easy to identify and delete test tags after CI runs.

## License

MIT
