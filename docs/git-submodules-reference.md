# Git Submodules Quick Reference

Quick commands for working with the TinyBigCorp multi-repo setup.

---

## Initial Setup

```bash
# Clone with submodules
git clone --recursive https://github.com/CkoThuw11/TinyBigCorp.git

# Or if already cloned
git submodule update --init --recursive
```

---

## Daily Development

### Working on Backend

```bash
cd backend
git checkout -b feature/my-feature
# Make changes
git add .
git commit -m "feat: my feature"
git push origin feature/my-feature
# Create PR in backend repo
```

### Working on Frontend

```bash
cd frontend
git checkout -b feature/my-feature
# Make changes
git add .
git commit -m "feat: my feature"
git push origin feature/my-feature
# Create PR in frontend repo
```

---

## Updating Submodules

### Pull Latest Changes

```bash
# Update all submodules to latest
git submodule update --remote

# Or update specific submodule
git submodule update --remote backend
git submodule update --remote frontend
```

### After Submodule PRs Merge

```bash
# From main repo root
git submodule update --remote backend frontend
git add backend frontend
git commit -m "chore: update submodules to latest"
git push
```

---

## Common Issues

### Submodule in Detached HEAD State

```bash
cd backend  # or frontend
git checkout main
git pull
```

### Submodule Not Updating

```bash
# Force update
git submodule update --init --force --remote
```

### Reset Submodule to Committed Version

```bash
git submodule update --init
```

---

## Useful Commands

```bash
# Check submodule status
git submodule status

# See which commit each submodule is at
git submodule

# Run command in all submodules
git submodule foreach 'git pull origin main'

# Clone and immediately update submodules
git clone --recurse-submodules https://github.com/CkoThuw11/TinyBigCorp.git
```

---

## Full Documentation

- [Main CONTRIBUTING.md](../CONTRIBUTING.md)
- [Backend CONTRIBUTING.md](../backend/CONTRIBUTING.md)
- [Frontend CONTRIBUTING.md](../frontend/CONTRIBUTING.md)
