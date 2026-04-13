# Implementation Plan: Fix Race Condition in fbc-fips-check-oci-ta

## Issue Context
The `fbc-fips-check-oci-ta` task fails inconsistently in parallel pipeline runs (e.g., OCP 4.20 vs 4.21). The failure signature is a missing file error (`lstat /tekton/home/unpacked-.../etc/redhat-release: no such file or directory`) during the `check-payload` step.

## Root Cause Analysis
The race condition occurs within the `fips-operator-check-step-action.yaml` step action. The script processes multiple `relatedImages` in parallel using background jobs. The working directories and image artifacts are named based on image labels: `component_label`, `version_label`, and `release_label`. 

However, multiple images can share the same labels (for example, different architecture manifests of the same image build). When these are processed concurrently, multiple background jobs attempt to write to, read from, and clean up the exact same `/tekton/home/unpacked-${component_label}-${version_label}-${release_label}` directory and `/tekton/home/${component_label}-${version_label}-${release_label}:latest` OCI image. 

If one job finishes or fails while another is still running `check-payload`, its `cleanup_image_artifacts` function deletes the shared directory, causing the other job to fail with a "no such file or directory" error.

## Proposed Solution
To ensure each background job operates in complete isolation, we need to append a unique identifier to the artifact paths. The `process_image` function already receives a unique `image_num` parameter.

### Changes Required
1. **Update `stepactions/fips-operator-check-step-action/0.1/fips-operator-check-step-action.yaml`:**
   - Modify the `cleanup_image_artifacts` function to accept `image_num` as a fourth parameter.
   - Update all references to the OCI image and unpacked directory paths to include `${image_num}`.
   - For example:
     - OCI Image: `/tekton/home/${component_label}-${version_label}-${release_label}-${image_num}:latest`
     - Unpacked Dir: `/tekton/home/unpacked-${component_label}-${version_label}-${release_label}-${image_num}`
     - Report File: `/tekton/home/report-${component_label}-${version_label}-${release_label}-${image_num}.csv`

2. **Testing:**
   - Validate the `fips-operator-check-step-action` by running `hack/test-build.sh` or the local task test script.
   - Verify that concurrent runs with identical component labels do not interfere with each other.
