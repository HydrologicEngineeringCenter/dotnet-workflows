# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository contains reusable GitHub Actions workflows for .NET projects in the HydrologicEngineeringCenter organization. It is not a .NET project itself—it provides CI/CD pipeline definitions that other .NET repositories consume.

## Repository Structure

```
.github/workflows/
├── integration.yml  # PR validation (build + optional tests)
├── snapshot.yml     # Dev package publishing on main branch push
└── release.yml      # Release package publishing on version tags
```

## Workflow Architecture

All workflows use `workflow_call` trigger (reusable workflows) and run on `windows-latest` runners.

**Common Pattern:**
1. Checkout with `fetch-depth: 0` (full history for versioning)
2. Setup .NET SDK (defaults to 9.0.x)
3. Build in Release configuration
4. Optional test execution
5. Pack and publish (snapshot/release only)

**Versioning Strategy:**
- **Snapshot**: Builds to `./build-output`, extracts version from each project's DLL individually, appends `.<run_number>-dev` suffix. Supports multiple projects via comma-separated `project-names` input.
- **Release**: Strips `v` prefix from git tag (e.g., `v1.2.3` → `1.2.3`). All projects use the same tag version via comma-separated `project-names` input.

**NuGet Publishing:**
- Requires `nuget-source` input (no default)
- Requires `NUGET_API_KEY` secret
- Uses `--skip-duplicate` to avoid conflicts

## Testing Changes

Since these are reusable workflows, test by:
1. Creating a test branch in a consuming .NET repository
2. Pointing the workflow reference to your branch: `@your-branch` instead of `@main`
3. Triggering the appropriate event (PR, push to main, or tag push)
