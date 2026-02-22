# .github

Organization-wide GitHub configuration, reusable workflows, and shared templates.

## Reusable Workflows

All reusable workflows live in `.github/workflows/` and are referenced as:

```
The-Cloud-Clock-Work/.github/.github/workflows/<name>@main
```

| Workflow | Purpose | Key Inputs | Secrets |
|----------|---------|------------|---------|
| `claude-reusable.yml` | Claude Code Action (issue/PR assistant) | `model`, `trigger_phrase`, `allowed_tools`, `timeout_minutes` | `ANTHROPIC_AUTH_TOKEN` (required), `ANTHROPIC_BASE_URL` (required) |
| `release-reusable.yml` | Semver bump → commit → tag → push | `bump` (required), `branch`, `version_file` | `RELEASE_TOKEN` (optional) |
| `ghcr-build-reusable.yml` | Build + push Docker image to GHCR | `dockerfile`, `context` | _(uses `GITHUB_TOKEN`)_ |
| `docker-publish-reusable.yml` | Build + push Docker image to DockerHub | `image_name` (required), `context` | `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN` |
| `pypi-publish-reusable.yml` | Build + publish to PyPI via OIDC | `python_version` | _(OIDC trusted publishers)_ |
| `docs-audit-reusable.yml` | Claude-powered docs audit with MCP | `scope`, `audit_script`, `anthropic_url_base` (required), + more | `ANTHROPIC_TOKEN_BASE`, `RELEASE_TOKEN`, `MCP_API_KEY` |

## Workflow Templates

| Template | Description |
|----------|-------------|
| `workflow-templates/claude-code.yml` | Drop-in Claude Code Action caller |

## Usage Examples

### Release

```yaml
# .github/workflows/release.yml
name: Release
on:
  workflow_dispatch:
    inputs:
      bump:
        description: "Version bump type"
        required: true
        type: choice
        options: [patch, minor, major]

permissions:
  contents: write

jobs:
  release:
    uses: The-Cloud-Clock-Work/.github/.github/workflows/release-reusable.yml@main
    with:
      bump: ${{ inputs.bump }}
    secrets:
      RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### GHCR Build

```yaml
# .github/workflows/build.yml
name: Docker Build
on:
  push:
    tags: ["v*"]

permissions:
  contents: read
  packages: write

jobs:
  build:
    uses: The-Cloud-Clock-Work/.github/.github/workflows/ghcr-build-reusable.yml@main
```

### DockerHub Publish

```yaml
# .github/workflows/docker-publish.yml
name: Publish Docker image
on:
  push:
    tags: ["v*"]

jobs:
  push:
    uses: The-Cloud-Clock-Work/.github/.github/workflows/docker-publish-reusable.yml@main
    with:
      image_name: my-project
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### PyPI Publish

```yaml
# .github/workflows/publish-pypi.yml
name: Publish to PyPI
on:
  push:
    tags: ["v*"]

permissions:
  contents: read
  id-token: write

jobs:
  publish:
    uses: The-Cloud-Clock-Work/.github/.github/workflows/pypi-publish-reusable.yml@main
```

### Docs Audit

```yaml
# .github/workflows/docs-audit.yml
name: Docs Audit
on:
  workflow_dispatch:
    inputs:
      scope:
        description: "Audit scope"
        required: false
        type: string

permissions:
  contents: write
  pull-requests: write

jobs:
  docs-audit:
    uses: The-Cloud-Clock-Work/.github/.github/workflows/docs-audit-reusable.yml@main
    with:
      scope: ${{ inputs.scope }}
      anthropic_url_base: ${{ vars.ANTHROPIC_URL_BASE }}
      anthropic_url_fallback: ${{ vars.ANTHROPIC_URL_FALLBACK }}
      anthropic_model_base: ${{ vars.ANTHROPIC_MODEL_BASE }}
      anthropic_model_fallback: ${{ vars.ANTHROPIC_MODEL_FALLBACK }}
      mcp_server_url: ${{ vars.MCP_SERVER_URL }}
    secrets:
      RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
      ANTHROPIC_TOKEN_BASE: ${{ secrets.ANTHROPIC_TOKEN_BASE }}
      ANTHROPIC_TOKEN_FALLBACK: ${{ secrets.ANTHROPIC_TOKEN_FALLBACK }}
      MCP_API_KEY: ${{ secrets.MCP_API_KEY }}
```
