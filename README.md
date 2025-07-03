# GitHub Actions Docker Private Registry Authentication Issues

This repository demonstrates authentication issues when using GitHub Actions with Docker actions and private registries. The tests showcase when Docker actions fail unexpectedly and when they succeed for different action types.

## Problem Overview

GitHub Actions handle Docker authentication inconsistently when using private registries, causing some action types to fail while others work perfectly. This affects enterprise users with private registries (ACR, ECR, GHCR, etc.) and forces the use of inefficient workarounds.

## What We're Testing

### ✅ **What Works Consistently**

- **Composite actions** using `docker run` commands
- **Local action calls** after `actions/checkout`
- **Direct Docker commands** in workflow steps

### ❌ **What Fails Unexpectedly**  

- **Docker image actions** (`using: "docker"` with `image: "docker://registry/image"`)
- **Remote action calls** without checkout
- **Dockerfile actions** with private base images (inconsistent behavior)

## Test Scenarios

Each test demonstrates the authentication behavior across three Docker action types:

### 1. **Composite Actions** (`using: "composite"`)

- **Expected**: ✅ Always succeeds
- **Reason**: Uses direct `docker run` commands which respect authentication

### 2. **Dockerfile Actions** (`using: "docker"`, `image: "Dockerfile"`)

- **Remote call**: ❌ Fails (no access to authentication)
- **Local call**: ✅ Succeeds (after `actions/checkout`)
- **Reason**: Inconsistent authentication propagation

### 3. **Image Actions** (`using: "docker"`, `image: "docker://registry/image"`)

- **Remote call**: ❌ Fails (cannot pull private image)
- **Local call**: ✅ Succeeds (after pre-pulling image)
- **Reason**: Docker authentication doesn't propagate to action runner

## Repository Structure

### Test Actions

```
test-gh-docker/
├── composite/          # 🧩 Composite action (always works)
├── dockerfile/         # 🐳 Dockerfile action (works locally)
└── image/             # 📦 Image action (works with workarounds)

common/
└── docker-registry-login/  # 🔐 Shared login action
```

### Workflows

```
.github/workflows/
├── orchestrator.yml         # 🎯 Calls other workflows (organized)
├── comprehensive.yml        # 📋 All tests in one file (monolithic)
├── composite.yml            # 🧩 Individual: Composite tests
├── dockerfile.yml           # 🐳 Individual: Dockerfile tests  
├── image.yml                # 📦 Individual: Image tests
└── dind.yml                # 🔄 Individual: Docker-in-Docker tests
```

## Usage Patterns

### **For Development & Debugging**

Use individual workflows for focused testing:

```bash
# Test specific action type
gh workflow run composite.yml
gh workflow run dockerfile.yml
gh workflow run image.yml
gh workflow run dind.yml
```

### **For Organized Testing**

Use the orchestrator for parallel execution:

```bash
gh workflow run orchestrator.yml
```

### **For Comprehensive Demonstration**

Use the comprehensive workflow to show all issues in one place:

```bash
gh workflow run comprehensive.yml
```

## Example Results

After running `docker/login-action` to authenticate with a private registry:

```yaml
# ❌ This fails - Docker action cannot access authentication
- uses: my-org/my-docker-action@main  # uses: docker, image: docker://private-registry/image

# ✅ This works - Local call with checkout
- uses: actions/checkout@v4
- uses: ./my-docker-action            # Same action, called locally

# ✅ This works - Composite action with docker run
- uses: my-org/my-composite-action@main  # uses: composite, runs: docker run private-registry/image
```

## Root Cause

Docker authentication from `docker/login-action` doesn't properly propagate to GitHub Actions' Docker action runner, but it does work for direct `docker` commands in composite actions and workflow steps.

## Current Workarounds

1. **Use `actions/checkout` before calling Docker actions locally**
2. **Pre-pull images with `docker pull`**
3. **Convert to composite actions using `docker run`**
4. **Use Docker-in-Docker containers**
5. **Make images public** (not feasible for enterprise)

## Expected Behavior

Docker authentication should work consistently across all action types (`using: "docker"` and `using: "composite"`).

## Impact

This affects enterprise users with private registries and forces:

- Inefficient workarounds
- Inconsistent action behavior
- Inability to use `using: "docker"` actions with private registries
- Additional complexity in CI/CD pipelines

## Environment

- **Runners**: `ubuntu-latest`
- **Registries**: Tested with ACR, GHCR, ECR
- **Authentication**: `docker/login-action@v3`
- **GitHub Token**: `GITHUB_TOKEN` with `packages: read` permission

## Contributing

To reproduce these issues:

1. **Fork this repository**
2. **Push Docker images to your private registry**
3. **Update registry URLs in workflows**
4. **Run the workflows and observe the authentication failures**

The failing tests demonstrate the core issue, while the succeeding tests show the necessary workarounds.

---

**Related Issues**: [GitHub Community Discussion](https://github.com/orgs/community/discussions) • [Docker/login-action](https://github.com/docker/login-action) • [Actions/runner](https://github.com/actions/runner)
