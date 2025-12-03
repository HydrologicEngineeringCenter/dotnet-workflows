# dotnet-workflows

Reusable GitHub Actions workflows for .NET projects in the HydrologicEngineeringCenter organization.

## Available Workflows

### Integration (`integration.yml`)

Runs on pull requests to validate builds and optionally run tests.

**Inputs:**
| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `dotnet-version` | string | No | `9.0.x` | .NET SDK version |
| `warnings-as-errors` | boolean | No | `false` | Treat compiler warnings as errors |
| `run-tests` | boolean | No | `false` | Run tests after build |
| `project-names` | string | No | `` | Comma-separated list of project names to build/test (if empty, builds entire solution) |

**Usage:**
```yaml
name: Integration
on:
  pull_request:
    branches: [ main ]
jobs:
  ci:
    uses: HydrologicEngineeringCenter/dotnet-workflows/.github/workflows/integration.yml@main
    with:
      run-tests: true
      warnings-as-errors: true
```

### Snapshot (`snapshot.yml`)

Builds and publishes dev packages on push to main branch.

**Inputs:**
| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `dotnet-version` | string | No | `9.0.x` | .NET SDK version |
| `project-names` | string | **Yes** | - | Comma-separated list of project names to pack |
| `version` | string | No | `0.0.0` | Base version for snapshot packages (final version: `<version>.<run_number>-dev`) |
| `run-tests` | boolean | No | `false` | Run tests after build |
| `nuget-source` | string | **Yes** | - | NuGet repository URL to publish packages to |

**Secrets:**
- `NUGET_API_KEY` - Required for publishing to NuGet repository

**Usage:**
```yaml
name: Snapshot
on:
  push:
    branches: [ main ]
jobs:
  snapshot:
    uses: HydrologicEngineeringCenter/dotnet-workflows/.github/workflows/snapshot.yml@main
    with:
      project-names: MyProject
      version: '1.0.0'
      nuget-source: https://nuget.example.com/repository
      run-tests: true
    secrets: inherit
```

**Multiple projects:**
```yaml
with:
  project-names: ProjectA, ProjectB, ProjectC
  version: '2.1.0'
  nuget-source: https://nuget.example.com/repository
```

### Release (`release.yml`)

Builds and publishes release packages when version tags are pushed.

**Inputs:**
| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `dotnet-version` | string | No | `9.0.x` | .NET SDK version |
| `project-names` | string | **Yes** | - | Comma-separated list of project names to pack (all use version from tag) |
| `run-tests` | boolean | No | `false` | Run tests after build |
| `nuget-source` | string | **Yes** | - | NuGet repository URL to publish packages to |

**Secrets:**
- `NUGET_API_KEY` - Required for publishing to NuGet repository

**Usage:**
```yaml
name: Release
on:
  push:
    tags: [ "v*.*.*" ]
jobs:
  release:
    uses: HydrologicEngineeringCenter/dotnet-workflows/.github/workflows/release.yml@main
    with:
      project-names: MyProject
      nuget-source: https://nuget.example.com/repository
      run-tests: true
    secrets: inherit
```

**Multiple projects:**
```yaml
with:
  project-names: ProjectA, ProjectB, ProjectC
  nuget-source: https://nuget.example.com/repository
```

## Examples

### Project with tests (e.g., ExpressionParser)
```yaml
# Integration.yml
uses: HydrologicEngineeringCenter/dotnet-workflows/.github/workflows/integration.yml@main
with:
  run-tests: true

# Snapshot.yml
uses: HydrologicEngineeringCenter/dotnet-workflows/.github/workflows/snapshot.yml@main
with:
  project-names: ExpressionParser
  version: '1.0.0'
  nuget-source: https://nuget.example.com/repository
  run-tests: true
secrets: inherit
```

### WPF project without tests (e.g., GenericControls)
```yaml
# Integration.yml
uses: HydrologicEngineeringCenter/dotnet-workflows/.github/workflows/integration.yml@main
with:
  warnings-as-errors: true

# Snapshot.yml
uses: HydrologicEngineeringCenter/dotnet-workflows/.github/workflows/snapshot.yml@main
with:
  project-names: GenericControls
  version: '1.0.0'
  nuget-source: https://nuget.example.com/repository
secrets: inherit
```

### Project using older .NET version
```yaml
uses: HydrologicEngineeringCenter/dotnet-workflows/.github/workflows/integration.yml@main
with:
  dotnet-version: '8.0.x'
  run-tests: true
```
