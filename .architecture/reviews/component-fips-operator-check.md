# Architecture Review: fips-operator-check-step-action

**Date**: 2026-03-26
**Review Type**: Component
**Reviewers**: Systems Architect, Security Specialist, Maintainability Expert, Performance Specialist, Pragmatic Enforcer, Bash Expert, Tekton Expert

## Executive Summary

The `fips-operator-check-step-action` component was reviewed following a recent bug fix addressing a race condition during parallel image processing. The fix successfully isolates concurrent background jobs by appending a unique `image_num` identifier to temporary directories and OCI image paths. This ensures that concurrent tasks do not interfere with each other's temporary artifacts.

**Overall Assessment**: Adequate

**Key Findings**:
- The race condition in parallel processing has been resolved by enforcing strict path isolation per job.
- The use of background jobs in bash to manage concurrency is lightweight but lacks robust error propagation or inter-process communication beyond simple file counters.
- Cleanup logic (`cleanup_image_artifacts`) is functional but relies on ignoring `rm` errors (`|| true`), which can mask deeper filesystem issues.

**Critical Actions**:
- Validate the fix against real-world pipeline runs with high concurrency.
- In the future, consider using Tekton's native matrix capabilities instead of managing parallel background tasks in a bash script if the complexity grows.

---

## System Overview

- **Component**: `fips-operator-check-step-action` (Tekton StepAction)
- **Scope of review**: Bash script implementation and concurrency model for parallel processing of related images.
- **Key technologies**: Bash, Tekton, Skopeo, Umoci, `check-payload`
- **Context**: The step action unpacks related images of operator bundle image builds and scans them for FIPS compliance. Due to parallel execution, a race condition occurred when identical component labels resulted in shared temporary directories.

---

## Individual Member Reviews

### Systems Architect

**Perspective**: Focuses on how components work together as a cohesive system and analyzes big-picture architectural concerns.

#### Key Observations
- The task implements its own parallel execution model using bash background jobs (`&`) and `wait`.
- The synchronization relies on polling (`jobs -r | wc -l`) and writing to a temporary directory (`counter_dir`).

#### Strengths
1. **Concurrency Limits**: The `MAX_PARALLEL` parameter prevents overloading the task pod's resources.

#### Concerns
1. **Custom Orchestration** (Impact: Medium)
   - **Issue**: Hand-rolling parallel task execution in bash instead of using Tekton's native features (like pipelines with matrices).
   - **Why it matters**: It obscures task observability and complicates error reporting in the Tekton UI.
   - **Recommendation**: Accept for now, but if the task grows more complex, consider breaking it down.

#### Recommendations
1. **Evaluate Tekton Matrix** (Priority: Medium, Effort: Large)
   - **What**: Evaluate replacing the internal bash loop with Tekton matrix features.
   - **Why**: Better native observability.
   - **How**: Prototype a matrix-based pipeline.

### Maintainability Expert

**Perspective**: Evaluates how well the architecture facilitates long-term maintenance, evolution, and developer understanding.

#### Key Observations
- The script is quite long and mixes several concerns (image inspection, OCI conversion, unpacking, payload checking).
- Variable scoping inside functions is correctly using `local`.

#### Strengths
1. **Targeted Fix**: The addition of `image_num` to artifacts is a minimal and highly effective fix for the race condition.

#### Concerns
1. **Error Handling Verbosity** (Impact: Medium)
   - **Issue**: Repeated error blocks and identical cleanup calls after every command failure.
   - **Why it matters**: Increases the script's length and maintenance burden.
   - **Recommendation**: Consolidate error handling into a standardized wrapper function.

#### Recommendations
1. **Refactor Error Handling** (Priority: Medium, Effort: Medium)
   - **What**: Create an error handling function that logs, cleans up, and increments the error counter.
   - **Why**: Reduces code duplication and prevents missed cleanups.
   - **How**: Abstract the repetitive failure block in `process_image`.

### Bash Expert

**Perspective**: Advocates for clean, straightforward scripts readable by human.

#### Key Observations
- Correct usage of `set -euo pipefail`.
- Uses `mapfile` safely for array creation.

#### Strengths
1. **Safe Variable Expansion**: Uses proper quoting to prevent globbing and word splitting.

#### Concerns
1. **Brittle Cleanup** (Impact: Low)
   - **Issue**: `rm -rf "..." 2>/dev/null || true` is used for cleanup.
   - **Why it matters**: While functional, it might hide unexpected permissions issues.
   - **Recommendation**: Use `if [ -e "..." ]; then rm -rf "..."; fi`.

#### Recommendations
1. **Improve Cleanup Predictability** (Priority: Low, Effort: Small)
   - **What**: Check file existence before deletion.
   - **Why**: Cleaner bash idiom.
   - **How**: Update `cleanup_image_artifacts`.

### Tekton Expert

**Perspective**: Advocates for clean, straightforward scripts readable by human (and proper Tekton integration).

#### Key Observations
- The step action uses `$(params.MAX_PARALLEL)` safely mapped to an environment variable `MAX_PARALLEL` rather than injecting it directly into the script.

#### Strengths
1. **Parameter Safety**: Adheres to the security rule of not injecting parameters directly into scripts.

#### Concerns
1. **Results Handling** (Impact: Low)
   - **Issue**: The script aggregates results in temporary files before echoing them to the Tekton results path.
   - **Why it matters**: It's a standard workaround, but can be fragile.

### Pragmatic Enforcer

**Perspective**: Rigorously questions whether proposed solutions, abstractions, and features are actually needed right now.

#### Key Observations
- The fix applied (`image_num`) is the simplest possible solution to the problem. It avoids over-engineering a complex locking mechanism.

#### Strengths
1. **Simplicity**: Did not introduce lock files, external dependencies, or complex IPC.

#### Concerns
1. **None**: The fix is pragmatic and adheres to YAGNI.

---

## Collaborative Discussion

**Systems Architect**: "The fix is sound, but we should be wary of building complex task orchestrators within bash scripts inside Tekton."

**Maintainability Expert**: "Agreed. The script is getting long. However, rewriting it entirely into a matrix pipeline would be a massive scope creep for this bug fix."

**Bash Expert**: "The bash code is solid. The background job polling loop (`while [ "$(jobs -r | wc -l)" -ge "${MAX_PARALLEL}" ]; do wait -n; done`) is actually quite elegant for pure bash."

**Pragmatic Enforcer**: "Let's stick to the current fix. It works, it's deployed, and it solves the immediate pain. We can document the desire for a Tekton matrix approach as technical debt, but we shouldn't act on it until this script becomes unmaintainable."

### Common Ground

The team agrees on:
1. The implementation successfully resolves the race condition.
2. The approach taken (appending an ID) is the simplest and most pragmatic solution.
3. The bash script is nearing the limits of what should be comfortably maintained without abstraction.

### Priorities Established

**Critical (Address Immediately)**:
1. None. The current fix is sufficient.

**Important (Address Soon)**:
1. Refactor error handling to reduce code duplication in `process_image`.

**Nice-to-Have (Consider Later)**:
1. Evaluate native Tekton Matrix for parallel execution.

---

## Consolidated Findings

### Strengths

1. **Pragmatic Bug Fix**: The race condition was fixed efficiently without introducing complex new dependencies or locking patterns.
2. **Bash Safety**: Excellent adherence to bash safety guidelines (`set -euo pipefail`, properly scoped variables).
3. **Tekton Security**: Safe parameter injection via environment variables.

### Areas for Improvement

1. **Error Handling Deduplication**:
   - **Current state**: Repetitive `echo`, `cleanup`, `return` blocks.
   - **Desired state**: A single reusable trap or error handling function.
   - **Gap**: Missing abstraction for error flows.
   - **Priority**: Medium
   - **Impact**: Improves maintainability and reduces script length.

### Technical Debt

**Medium Priority**:
- **Custom Concurrency in Bash**:
  - **Impact**: Makes execution flow harder to trace in Tekton UI.
  - **Resolution**: Evaluate Tekton's native matrix/fan-out capabilities.
  - **Effort**: Large
  - **Recommended Timeline**: When the task requires its next major functional update.

### Risks

**Technical Risks**:
- **Silent Failures in Cleanup** (Likelihood: Low, Impact: Low)
  - **Description**: Ignoring all errors during `rm -rf` might mask serious I/O issues on the underlying volume.
  - **Mitigation**: Add checks for file existence before deletion.
  - **Owner**: Task Maintainers.

---

## Recommendations

### Short-term (2-8 weeks)

1. **Refactor Script Error Handling**
   - **Why**: To improve readability and prevent future regressions where cleanup might be forgotten in a new error branch.
   - **How**: Introduce a helper function for the failure path in `process_image`.
   - **Owner**: Build Definitions Team.
   - **Success Criteria**: Script size reduced, logic deduplicated.
   - **Estimated Effort**: Small.

### Long-term (2-6 months)

1. **Investigate Tekton Matrix Migration**
   - **Why**: To align with Tekton native orchestration paradigms.
   - **How**: Create a spike to test a matrix-based pipeline implementation for FIPS checking.
   - **Owner**: Systems Architect.
   - **Success Criteria**: A working prototype that proves (or disproves) the viability of matrix execution for this specific workload.
   - **Estimated Effort**: Medium.

---

## Follow-up

**Tracking**: Create issues for the Short-term and Long-term recommendations if prioritized by the maintainers.

---

## Appendix

### Review Methodology

This review was conducted using the AI Software Architect framework. Each member reviewed independently, then collaborated to synthesize findings and prioritize recommendations.

**Pragmatic Mode**: Balanced
- All recommendations evaluated through YAGNI lens.

**Review Complete**