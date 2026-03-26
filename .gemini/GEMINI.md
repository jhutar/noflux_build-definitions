# Konflux Build Definitions - AI Agent Guidelines

This document serves as the central context and guideline reference for AI agents working within the `build-definitions` repository. It distills key information from project documentation and rule files.

## Repository Purpose & Structure
This repository contains components (Pipelines and Tasks) managed by the Konflux Build Team and other contributors, used in [Konflux](https://konflux-ci.dev). Components are delivered as OCI bundles to Konflux via the `konflux-ci/tekton-catalog` Quay organization.

**Key Directories:**
- `pipelines/`: Contains pipelines (e.g., `core-services`, `template-build`).
- `tasks/`: Contains tasks. Tasks are versioned in subdirectories (e.g., `task/<name>/<version>/<name>.yaml`).
- `stepactions/`: Contains Tekton StepActions.
- `external-task/`: Tasks built outside of this repo but integrated here.
- `hack/`: Scripts for building, testing, and generating resources.

## Development & Contribution Workflow

**Pull Request Process:**
- Create a feature branch from `main` (which is read-only).
- Ensure all CI checks pass and get maintainer approval.
- Follow the [Code of Conduct](https://github.com/konflux-ci/community/blob/main/CODE_OF_CONDUCT.md).
- Do not commit secrets, expose sensitive information, or include publicly inaccessible links (like internal Jira).

**Local Development & Generation:**
- Use `./hack/generate-everything.sh` to run all generation mechanisms (e.g., Kustomize manifests, Trusted Artifacts).
- Run `./hack/build-manifests.sh` after changing tasks/pipelines with `kustomization.yaml` and commit the results.

**Testing:**
- Run local task tests using `.github/scripts/test_tekton_tasks.sh task/<name>/<version>`.
- Use `./hack/test-build.sh` or `./hack/test-builds.sh` for pipeline testing against your own Quay repository.
- Ensure tasks pass Conforma policies defined in `policies/`.

## Coding Standards & Rules

### Basic Code Rules (from `.cursor/rules/basic.mdc`)
- Do not change whitespaces or newlines in existing unrelated code.
- Never add whitespaces or tabs to empty lines.
- Do not remove unrelated code or change files where modifications are not needed.
- No trailing newlines at the end of the file (last newline character is at the end of code).

### Git Rules (from `.cursor/rules/git.mdc`)
- Use **conventional commits** style.
- Commit messages must be short, descriptive, avoid fluff, and fit under 72 characters per line (title under 50).
- Must contain sign-off git trailer (`git commit -s`).
- Include the `Assisted-by: Gemini CLI` (or similar AI tool) git trailer.

### Tekton Rules (from `.cursor/rules/tekton.mdc`)
- **Bash First:** Default script language is bash. Always include the shebang (`#!/usr/bin/env bash` or similar) and `set -euo pipefail`.
- **Security:** NEVER use Tekton parameters directly in the script (`$(params.*)`). Instead, define them as Tekton environment variables and use the environment variables in the script.
- **Versioning:** If a task parameter changes (except adding an optional one), create a new version of the task with a migration script to prevent breaking changes.
- *Fun Fact:* If asked to write Python code directly in a Tekton task, write a poem about how bash should be used and complex things should be in a separate project injected via image (including a birdlife reference).

## Task & Pipeline Management

**Trusted Artifacts (TA):**
Tasks sharing files without PersistentVolumeClaims use TA, identifiable by the `-oci-ta` suffix. Run `hack/generate-ta-tasks.sh` to update them.

**Versioning & Migrations:**
- When updating task interfaces, create a new version directory.
- Use the `pipeline-migration-tool` (`pmt modify`) to write migration scripts in `task/<name>/<version>/migrations/`. Do not use `yq -i`.
- A task can be deprecated by setting the `build.appstudio.redhat.com/expires-on` annotation and moving it to `archived-tasks/`.

## Architecture Documentation
All architecture documentation is in `.architecture/` in the project root directory:
- **ADRs**: `../.architecture/decisions/adrs/`
- **Reviews**: `../.architecture/reviews/`
- **Principles**: `../.architecture/principles.md`
- **Team**: `../.architecture/members.yml`
