# ADR-001: Handle Missing redhat-release in FBC FIPS Check

## Status

Proposed

## Context

The `fbc-fips-check-oci-ta` task is failing inconsistently across different OpenShift (OCP) version pipelines (e.g., failing on 4.20, succeeding on 4.21) despite using identical operator images. The root cause is that the `check-payload` tool attempts to scan the unpacked image filesystem and crashes when it cannot find `/etc/redhat-release` (error: `lstat .../etc/redhat-release: no such file or directory`). 
Some operator plugin containers (like `network-observability-console-plugin-container`) do not include this file, potentially because they use minimalistic base images like `scratch` or `ubi-micro`. This causes pipeline builds to fail unpredictably.

## Decision Drivers

* **Pipeline Reliability**: The FBC FIPS check task needs to be stable across all OCP versions.
* **Compatibility**: Operator images might legitimately lack an `/etc/redhat-release` file if they are built from scratch or minimal base images.
* **Compliance**: `check-payload` is a mandatory step for ensuring FIPS compliance; it must not crash on structural assumptions.

## Decision

We will update the `fbc-fips-check-oci-ta` Tekton task to explicitly handle the missing `/etc/redhat-release` file prior to executing the `check-payload` scan. The script will be updated to check if the file exists within the unpacked image directory. If it is missing, the script will inject a dummy `/etc/redhat-release` file (e.g., containing "Red Hat Enterprise Linux release 9 (Plow)") into the unpacked filesystem to satisfy `check-payload`'s structural requirement, allowing the scan to proceed on the actual binaries.

**Architectural Components Affected:**
* Task: `fbc-fips-check-oci-ta` (specifically the step executing `check-payload`)

**Interface Changes:**
* None to the external task parameters.

## Consequences

### Positive

* The FIPS check task will no longer crash on minimal images lacking OS release files.
* Build pipeline stability will be restored across all OCP target versions.
* Operator teams will not be forced to change their base images solely to satisfy a scanning tool's file assumption.

### Negative

* Injecting a dummy `redhat-release` file is a workaround; ideally, `check-payload` should handle this gracefully itself.
* If `check-payload` behavior strictly relies on matching the exact OS version for specific vulnerability databases, a dummy file might lead to slightly inaccurate FIPS validation context (though the binary scanning should remain unaffected).

### Neutral

* Added a few lines of bash script to the task definition.

## Implementation Strategy

### Blast Radius

**Impact Scope**: Limited to the `fbc-fips-check-oci-ta` task execution.

**Affected Components**:
- `fbc-fips-check-oci-ta` - The task execution will be modified to add the pre-scan validation.

**Risk Mitigation**:
- We will only inject the file if it does not already exist. Existing valid OS files will be preserved.

### Reversibility

**Reversibility Level**: High

**Rollback Feasibility**:
- The change is confined to a bash script within a Tekton task. Reverting the task definition will restore the previous behavior.

### Sequencing & Timing

**System Readiness**:
- **Dependencies**: Tekton tasks already support running arbitrary bash commands prior to executing the main binary.

**Readiness Assessment**: Ready to implement.

## Alternatives Considered

### Alternative 1: Update `check-payload` upstream

Modify the `check-payload` tool to gracefully skip or warn when `/etc/redhat-release` is missing, rather than crashing.

**Pros:**
* Solves the root cause appropriately.
* Cleaner task definition.

**Cons:**
* Requires coordination with the upstream tool maintainers.
* Slower time-to-resolution for the currently broken pipelines.

### Alternative 2: Skip FIPS check for minimal images

Detect if the image is minimal and bypass the `check-payload` scan entirely.

**Pros:**
* No need to hack the filesystem.

**Cons:**
* Violates security policies; FIPS checks are mandatory for all executable payloads, regardless of the base image.

## Validation

**Acceptance Criteria:**
- [ ] The `fbc-fips-check-oci-ta` task successfully scans an image known to lack `/etc/redhat-release` (e.g., `network-observability-console-plugin-container`).
- [ ] Existing images with valid OS release files continue to be scanned successfully.

**Testing Approach:**
* Run local task tests using the provided script `.github/scripts/test_tekton_tasks.sh task/fbc-fips-check-oci-ta/<version>` with an image payload missing the release file.

## References

* GitHub Issue: https://github.com/nonflux/build-definitions/issues/1