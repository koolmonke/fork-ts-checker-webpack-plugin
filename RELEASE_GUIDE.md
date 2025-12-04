# Fork TS Checker Webpack Plugin - Release Guide

This guide explains how to release the fork-ts-checker-webpack-plugin package, which uses an automated release process powered by semantic-release.

## Overview

This project uses **semantic-release** to automate version management and package publishing. The release process is triggered automatically when commits are pushed to specific branches (`main`, `alpha`, `beta`).

## Prerequisites

### Required Permissions

To trigger releases, you need:

1. **Write access** to the repository
2. **GitHub Token** with appropriate permissions (automatically provided by GitHub Actions)
3. **NPM Token** with publish permissions for the `@koolmonke/fork-ts-checker-webpack-plugin` package

### Repository Secrets

The following secrets must be configured in the repository settings:

- `NPM_TOKEN`: Your npm authentication token with publish permissions
- `GITHUB_TOKEN`: Automatically provided by GitHub Actions (needs `contents: write`, `pull-requests: write`, `deployments: write` permissions)

### Development Environment Setup

```bash
# Clone the repository
git clone https://github.com/TypeStrong/fork-ts-checker-webpack-plugin.git
cd fork-ts-checker-webpack-plugin

# Install dependencies
yarn install

# Verify build works
yarn build

# Run tests
yarn test
```

## Automated Release Workflow

### How It Works

1. **CI/CD Pipeline**: When code is pushed to `main`, `alpha`, or `beta` branches, GitHub Actions triggers the release workflow
2. **Build & Test**: The code is built and tested on multiple Node.js versions and operating systems
3. **Semantic Release**: If tests pass, `semantic-release` analyzes commits to determine the version bump
4. **Package Publishing**: The package is automatically published to npm
5. **GitHub Release**: A GitHub release is created with changelog

### Release Branches

- `main`: Production releases (e.g., 1.0.0, 1.1.0, 1.1.1)
- `alpha`: Alpha pre-releases (e.g., 1.0.0-alpha.1)
- `beta`: Beta pre-releases (e.g., 1.0.0-beta.1)

### Version Determination

Version numbers are automatically determined based on commit messages:

| Commit Type        | Release Type | Example       |
| ------------------ | ------------ | ------------- |
| `feat:`            | Minor        | 1.0.0 → 1.1.0 |
| `fix:`             | Patch        | 1.0.0 → 1.0.1 |
| `perf:`            | Patch        | 1.0.0 → 1.0.1 |
| `refactor:`        | Patch        | 1.0.0 → 1.0.1 |
| `docs:`            | Patch        | 1.0.0 → 1.0.1 |
| `BREAKING CHANGE:` | Major        | 1.0.0 → 2.0.0 |

## Making Changes That Trigger Releases

### Commit Message Format

Use conventional commits to ensure proper version bumps:

```bash
# Feature
git commit -m "feat: add new TypeScript configuration option"

# Bug fix
git commit -m "fix: resolve path resolution issue on Windows"

# Breaking change
git commit -m "feat: redesign plugin API

BREAKING CHANGE: The plugin constructor now requires a configuration object"
```

### Using Commitizen (Recommended)

The project includes commitizen for standardized commit messages:

```bash
# Interactive commit wizard
yarn commit

# Or use git-cz directly
npx git-cz
```

### Commit Types

- `feat`: New features
- `fix`: Bug fixes
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `chore`: Build process, CI, or auxiliary tool changes
- `revert`: Reverting previous changes

## Manual Release Process

While the process is automated, you can manually trigger a release:

### Option 1: Push to Main Branch

```bash
# Make your changes
git add .
git commit -m "feat: add your feature"
git push origin main
```

### Option 2: Local Semantic Release (For Testing)

```bash
# Install semantic-release globally (if not already installed)
npm install -g semantic-release

# Run semantic-release locally (dry run)
semantic-release --dry-run

# Run actual release (use with caution)
semantic-release
```

⚠️ **Warning**: Running semantic-release locally will publish to npm if you have proper credentials configured. Use `--dry-run` first to test.

### Option 3: Local Release with npm (For Emergency Situations)

Yes, you can release locally using npm directly. This is useful for emergency releases or when the automated process isn't working:

```bash
# First, ensure you're logged in to npm
npm login

# Verify you have publish permissions for the package
npm access ls-collaborators @koolmonke/fork-ts-checker-webpack-plugin

# Build the project
yarn build

# Determine the next version (check current version in package.json)
npm view @koolmonke/fork-ts-checker-webpack-plugin version

# Bump the version (choose appropriate type)
npm version patch  # for bug fixes (1.0.0 → 1.0.1)
npm version minor  # for features (1.0.0 → 1.1.0)
npm version major  # for breaking changes (1.0.0 → 2.0.0)

# Publish to npm
npm publish --access public

# Push the version tag to GitHub
git push origin main --tags
```

**Important Notes for Local Releases:**

1. **Manual Chelog**: You'll need to manually update the CHANGELOG.md since semantic-release won't run
2. **GitHub Release**: Create the GitHub release manually through the GitHub UI
3. **Version Conflicts**: Be careful not to create version conflicts with future automated releases
4. **Use Sparingly**: This method should only be used for emergency situations

**To check if you can publish locally:**

```bash
# Check if you're logged in to npm
npm whoami

# Check package permissions
npm access ls-packages @koolmonke

# Test dry-run publish
npm publish --dry-run --access public
```

## Release Configuration

The release process is configured in:

1. [`release.config.js`](./release.config.js) - Semantic release configuration
2. [`changelog.config.js`](./changelog.config.js) - Commitizen/changelog configuration
3. [`.github/workflows/main.yml`](./.github/workflows/main.yml) - GitHub Actions workflow

### Key Configuration Details

- **Branches**: `main` (production), `alpha` and `beta` (pre-releases)
- **Plugins**:
  - `@semantic-release/commit-analyzer` - Analyzes commits for version bump
  - `@semantic-release/release-notes-generator` - Generates changelog
  - `@semantic-release/npm` - Publishes to npm
  - `@semantic-release/github` - Creates GitHub releases
  - `@semantic-release/exec` - Runs custom scripts

## Troubleshooting

### Release Failed

1. **Check GitHub Actions**: Go to the Actions tab in GitHub to see what failed
2. **Verify Tests**: Ensure all tests are passing locally with `yarn test`
3. **Check Build**: Verify the build succeeds with `yarn build`
4. **Review Commit Messages**: Ensure commits follow conventional format

### Common Issues

#### "No release" - No changes detected

**Cause**: No commits warrant a version bump (only chore, docs, or revert commits)

**Solution**: Add a feature or fix commit:

```bash
git commit -m "feat: add new feature"
git push origin main
```

#### "Permission denied" - NPM publish failed

**Cause**: Invalid or missing NPM_TOKEN

**Solution**:

1. Check that `NPM_TOKEN` is set in repository secrets
2. Verify the token has publish permissions for the package
3. Regenerate the token if necessary

#### "404 Not Found" - Package doesn't exist

**Cause**: Publishing to wrong package name or registry

**Solution**:

1. Verify `package.json` has correct name: `@koolmonke/fork-ts-checker-webpack-plugin`
2. Check that you're publishing to the correct npm registry

### Manual Version Bump (Emergency)

If you need to manually bump the version (emergency only):

```bash
# Install semver
npm install -g semver

# Get current version
CURRENT_VERSION=$(node -p "require('./package.json').version")
echo "Current version: $CURRENT_VERSION"

# Bump version (choose appropriate type)
NEW_VERSION=$(semver -i bump $CURRENT_VERSION)
echo "New version: $NEW_VERSION"

# Update package.json
npm version $NEW_VERSION --no-git-tag-version

# Commit and push
git add package.json
git commit -m "chore: bump version to $NEW_VERSION"
git tag v$NEW_VERSION
git push origin main --tags
```

**Alternative: Using npm version command**

```bash
# Build the project first
yarn build

# Use npm to version and publish in one step
npm version patch -m "chore: bump version to %s"
npm publish --access public

# Push tags to GitHub
git push origin main --follow-tags
```

## Best Practices

1. **Write Clear Commit Messages**: Use conventional commit format for proper versioning
2. **Test Thoroughly**: Ensure all tests pass before pushing to main
3. **Use Feature Branches**: Develop on feature branches, then merge to main
4. **Monitor Releases**: Keep an eye on automated releases in GitHub Actions
5. **Document Breaking Changes**: Clearly document any breaking changes in commit messages

## Post-Release Checklist

After a successful release:

- [ ] Verify the package is available on npm
- [ ] Check the GitHub release was created
- [ ] Update documentation if needed
- [ ] Monitor for any issues reported by users
- [ ] Consider announcing significant releases

## Getting Help

If you encounter issues with the release process:

1. Check the [GitHub Actions documentation](https://docs.github.com/en/actions)
2. Review the [semantic-release documentation](https://semantic-release.gitbook.io/)
3. Open an issue in the repository with details about the problem
4. Contact the repository maintainers for urgent issues
