# scripts/

Helper scripts for the homelab SOC.

## Secret scanning — gitleaks

This repo uses [gitleaks](https://github.com/gitleaks/gitleaks) for pre-commit secret
scanning. The config is in `.gitleaks.toml`.

### Install gitleaks (macOS)
```sh
brew install gitleaks
```

### Install gitleaks (Linux)
```sh
# Download the latest release binary
curl -sSfL https://github.com/gitleaks/gitleaks/releases/latest/download/gitleaks_linux_amd64.tar.gz \
  | tar -xz -C /usr/local/bin gitleaks
```

### Manual scan (before pushing if hook is not installed)
```sh
gitleaks protect --staged
```

### Verify the pre-commit hook is active
```sh
cat .git/hooks/pre-commit
```

The hook exits non-zero and blocks the commit if any secrets are found.
If gitleaks is not installed, the hook prints a warning but does not block the commit —
install gitleaks to enforce scanning.
