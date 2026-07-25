<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-plan/v5.7.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-plan/v5.7.1** was hardened automatically. 40 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml interpolate ${{ }} expressions directly into shell commands (rule a), allowing script injection. Affected steps and examples:

1. 'Set atmos cli config path vars' (line ~91): `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — attacker-controlled input interpolated directly into shell.

2. 'Add Terraform and OpenTofu to Aqua' (line ~196): `if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).terraform-version }}" != "" ...` and `VERSION="${{ fromJson(steps.atmos-settings.outputs.settings).terraform-version }}"` — step outputs interpolated directly into shell.

3. 'Set atmos cli base path vars' (line ~258): `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` — step output interpolated directly into shell.

4. 'Define Job Variables' (line ~277): `STACK_NAME=$(echo "${{ inputs.stack }}" | sed ...)`, `COMPONENT_PATH=${{ fromJson(steps.atmos-settings.outputs.settings).component-path }}`, `COMPONENT_NAME=$(echo "${{ inputs.component }}" | sed ...)`, `PLAN_FILE=... ${{ inputs.sha }}.planfile` — multiple inputs and step outputs interpolated directly into shell.

5. 'Atmos Terraform Plan' (line ~338): Numerous direct interpolations including `${{ inputs.identity }}`, `${{ inputs.atmos-pro-upload-status }}`, `${{ github.repository_owner }}`, `${{ github.event.repository.name }}`, `${{ inputs.component }}`, `${{ inputs.stack }}`, `${{ inputs.sha }}`, `${{ inputs.debug }}`, `${{ inputs.pr-comment }}`, `${{ inputs.drift-detection-mode-enabled }}` — all interpolated directly into shell commands passed to tfcmt and atmos.

6. 'Set Plan Results' (line ~568): `echo "plan_file=${{ steps.vars.outputs.plan_file }}" >> $GITHUB_OUTPUT` — step output interpolated directly into shell.

7. 'Store Component Metadata to Artifacts' (line ~636): `echo -n '{ "stack": "${{ inputs.stack }}", "component": "${{ inputs.component }}", ... }'` — inputs interpolated directly into shell.

8. 'Publish Summary or Generate GitHub Issue Description' (line ~655): `if [[ "${{ inputs.drift-detection-mode-enabled }}" == "true" ]]` — input interpolated directly into shell.

All ${{ }} expressions must be moved to env: blocks and the shell variables must be double-quoted.

Locations:

- `action.yml:91`
- `action.yml:196`
- `action.yml:258`
- `action.yml:277`
- `action.yml:338`
- `action.yml:568`
- `action.yml:636`
- `action.yml:655`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs and step outputs to $GITHUB_ENV or $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. 'Set atmos cli config path vars' (line ~91): `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — the user-controlled input `inputs.atmos-config-path` is written directly to GITHUB_ENV. A newline in the value would allow injecting arbitrary environment variables.

2. 'Set atmos cli base path vars' (line ~258): `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` then `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV` — the step output (which is ultimately derived from workflow-controlled atmos settings) is written to GITHUB_ENV without sanitization.

3. 'Define Job Variables' (line ~277): Multiple values derived from `inputs.stack`, `inputs.component`, `inputs.sha`, and `steps.atmos-settings.outputs.settings` are written to $GITHUB_OUTPUT (e.g., `echo "stack_name=${STACK_NAME}" >> $GITHUB_OUTPUT`, `echo "plan_file=${PLAN_FILE}" >> $GITHUB_OUTPUT`) without sanitization. A newline in any of these values would allow injecting arbitrary output variables.

Locations:

- `action.yml:91`
- `action.yml:258`
- `action.yml:277`

### unpinned-uses (severity: high)

Multiple uses: references in action.yml use mutable tags instead of full 40-character commit SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised.

Unpinned references found:
- `actions/checkout@v4` (line ~87)
- `cloudposse/github-action-setup-atmos@v2` (line ~95)
- `cloudposse/github-action-atmos-get-setting@v2` (line ~100)
- `aws-actions/configure-aws-credentials@v4` (line ~247, line ~481)
- `actions/cache@v4` (line ~330)
- `cloudposse/github-action-terraform-plan-storage@v1` (line ~503, line ~527)
- `infracost/actions/setup@v3` (line ~561)
- `actions/upload-artifact@v4` (line ~686)

Note: `actions/cache@5a3ec84eff668545956fd18022155c47e93e2684` and `aquaproj/aqua-installer@5e54e5cee8a95ee2ce7c04cb993da6dfad13e59c` are correctly pinned to full SHAs.

Locations:

- `action.yml:87`
- `action.yml:95`
- `action.yml:100`
- `action.yml:247`
- `action.yml:330`
- `action.yml:481`
- `action.yml:503`
- `action.yml:527`
- `action.yml:561`
- `action.yml:686`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.atmos-config-path }}" appears directly in run: block of step "Set atmos cli config path vars"; move to env: map

Locations:

- `action.yml:105`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.debug }}" appears directly in run: block of step "Add Terraform and OpenTofu to Aqua"; move to env: map

Locations:

- `action.yml:235`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:287`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:290`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:293`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:294`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.identity }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:350`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.identity }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:351`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.atmos-pro-upload-status }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:356`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:365`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:367`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:368`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-image }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:370`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-url }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:371`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.drift-detection-mode-enabled }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:373`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pr-comment }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:374`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.debug }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:375`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pr-comment }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:377`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:379`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:380`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pr-comment }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:391`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:397`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:399`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:400`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-image }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:402`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-url }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:403`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.drift-detection-mode-enabled }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:405`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.debug }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:407`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.drift-detection-mode-enabled }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:415`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:544`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:544`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:549`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:549`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Store Component Metadata to Artifacts"; move to env: map

Locations:

- `action.yml:585`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Store Component Metadata to Artifacts"; move to env: map

Locations:

- `action.yml:585`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.drift-detection-mode-enabled }}" appears directly in run: block of step "Publish Summary or Generate GitHub Issue Description for Drift Detection"; move to env: map

Locations:

- `action.yml:593`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.drift-detection-mode-enabled }}" appears directly in run: block of step "Publish Summary or Generate GitHub Issue Description for Drift Detection"; move to env: map

Locations:

- `action.yml:611`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed all security findings in hardened/action/action.yml:

1. UNPINNED-USES: Pinned all 8 unpinned action references to full 40-character commit SHAs: actions/checkout@v4, cloudposse/github-action-setup-atmos@v2, cloudposse/github-action-atmos-get-setting@v2, aws-actions/configure-aws-credentials@v4 (×2), actions/cache@v4, cloudposse/github-action-terraform-plan-storage@v1 (×2), infracost/actions/setup@v3, actions/upload-artifact@v4.

2. SCRIPT-INJECTION / STATIC-INLINE-INJECTION: Moved ALL ${{ }} expressions out of run: blocks into env: blocks for every affected step. No ${{ inputs.* }}, ${{ fromJson(...) }}, ${{ github.* }}, or ${{ steps.*.outputs.* }} expressions remain in any run: block.

3. GITHUB-ENV-INJECTION: Added printf '%s' ... | tr -d '\n\r' sanitization for all values derived from user-controlled inputs before writing to $GITHUB_ENV or $GITHUB_OUTPUT. The 'Define Job Variables' step now sanitizes all 12 output values before writing them. The 'Set atmos cli config path vars' and 'Set atmos cli base path vars' steps sanitize before writing to GITHUB_ENV. The 'Set Plan Results' step sanitizes before writing to GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all three findings:

1. **script-injection (action.yml)**: Converted `base_cmd` and `atmos_pro_flags` from plain string variables to bash arrays. They are now initialized as `base_cmd=()` and `atmos_pro_flags=()`, elements appended with `+=("value")`, and expanded safely with `"${base_cmd[@]}"` and `"${atmos_pro_flags[@]}"`.

2. **script-injection (workflow files)**: Fixed in all 11 test workflow files by moving `${{ runner.temp }}` and `${{ env.AWS_REGION }}` expressions from `run:` shell strings into `env:` blocks as `RUNNER_TEMP` and `AWS_REGION_VAL`. Shell scripts now reference `$RUNNER_TEMP` and `${AWS_REGION_VAL}`.

3. **unpinned-uses**: Pinned all mutable action references to full commit SHAs:
   - `actions/checkout@v4` → `11d5960a326750d5838078e36cf38b85af677262`
   - `nick-fields/assert-action@v2` → `aa0067e01f0f6545c31755d6ca128c5a3a14f6bf`
   - `kibertoad/wait-action@1.0.1` → `99f6f101c5be7b88bb9b41c0d3b810722491b8e5`
   - `actions/download-artifact@v4` → `d3f86a106a0bac45b974a628896c90dbdf5c8093`
   - `cloudposse/.github` shared workflows `@main` → `8244c7c9142e92281e7841f655fa48e9ceb9b454`

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in all 11 workflow files (.github/workflows/integration-tests.yml, test-atmos-pro-enabled.yml, test-changes-exists-drift.yml, test-changes-exists.yml, test-failed-plan-drift.yml, test-failed-plan.yml, test-infra-cost.yml, test-no-changes-drift-more.yml, test-no-changes.yml, test-settings-action-disabled-drift.yml, test-settings-action-disabled.yml). In each file, the ${{ secrets.TERRAFORM_STATE_BUCKET }}, ${{ secrets.TERRAFORM_STATE_TABLE }}, ${{ secrets.TERRAFORM_STATE_ROLE }}, and ${{ secrets.TERRAFORM_PLAN_ROLE }} expressions were moved from run: shell blocks into the step's env: block as TERRAFORM_STATE_BUCKET, TERRAFORM_STATE_TABLE, TERRAFORM_STATE_ROLE, and TERRAFORM_PLAN_ROLE environment variables. The shell scripts were updated to reference these as ${TERRAFORM_STATE_BUCKET}, ${TERRAFORM_STATE_TABLE}, ${TERRAFORM_STATE_ROLE}, and ${TERRAFORM_PLAN_ROLE} respectively. The sed commands were also changed from single-quoted to double-quoted strings to allow shell variable expansion.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the 'Atmos Terraform Plan' step of action.yml. The $VARS_SUMMARY_FILE variable was expanded unquoted inside a command substitution: `$([[ "$INPUT_PR_COMMENT" == "false" ]] && echo "--output $VARS_SUMMARY_FILE")`. Changed to `$([[ "$INPUT_PR_COMMENT" == "false" ]] && echo "--output \"$VARS_SUMMARY_FILE\"")` so the path is properly quoted, preventing word-splitting and potential command injection from user-controlled values in inputs.stack and inputs.component.

### Iteration 5

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in hardened/action/action.yml:
1. Line ~370: Quoted `$ATMOS_COMMAND` → `"$ATMOS_COMMAND"` to prevent shell metacharacter injection from the workflow-controllable `steps.atmos-settings.outputs.settings.command` value.
2. Line ~315: Replaced the unquoted command substitution `$([[ "$INPUT_PR_COMMENT" == "false" ]] && echo "--output \"$VARS_SUMMARY_FILE\"")` with a pre-built bash array `output_flags` that safely passes `--output "$VARS_SUMMARY_FILE"` as separate properly-quoted arguments. Also replaced the `-patch` and log-level inline command substitutions with proper bash conditionals and variables for consistency and safety.

