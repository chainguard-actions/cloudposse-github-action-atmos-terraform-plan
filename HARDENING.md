<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-plan/v5.7.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-plan/v5.7.1** was hardened automatically. 40 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple uses: references in action.yml use mutable version tags instead of pinned full 40-character SHA digests, making the action vulnerable to supply-chain attacks. Unpinned references: actions/checkout@v4, cloudposse/github-action-setup-atmos@v2, cloudposse/github-action-atmos-get-setting@v2, aws-actions/configure-aws-credentials@v4 (appears twice), actions/cache@v4, cloudposse/github-action-terraform-plan-storage@v1 (appears twice), infracost/actions/setup@v3, actions/upload-artifact@v4. Correctly pinned: actions/cache@5a3ec84eff668545956fd18022155c47e93e2684 and aquaproj/aqua-installer@5e54e5cee8a95ee2ce7c04cb993da6dfad13e59c.

Locations:

- `action.yml:88`
- `action.yml:95`
- `action.yml:102`
- `action.yml:270`
- `action.yml:284`
- `action.yml:371`
- `action.yml:556`
- `action.yml:590`
- `action.yml:637`
- `action.yml:762`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions inside shell command strings (sub-rule a), allowing shell metacharacter injection. Affected steps: (1) 'Set atmos cli config path vars': echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV — inputs.atmos-config-path interpolated directly into shell. (2) 'Set atmos cli base path vars': ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}" — step output interpolated directly. (3) 'Define Job Variables': STACK_NAME=$(echo "${{ inputs.stack }}" | sed ...), COMPONENT_PATH=${{ fromJson(...).component-path }}, COMPONENT_NAME=$(echo "${{ inputs.component }}" | sed ...), PLAN_FILE=.../$COMPONENT_SLUG-${{ inputs.sha }}.planfile — multiple inputs and step outputs interpolated directly. (4) 'Atmos Terraform Plan': if [[ -n "${{ inputs.identity }}" ]], base_cmd+=" --identity=${{ inputs.identity }}", atmos terraform plan ${{ inputs.component }}, --stack ${{ inputs.stack }}, -owner "${{ github.repository_owner }}", -repo "${{ github.event.repository.name }}", ${{ fromJson(...).command }} show -json — attacker-controlled values interpolated directly into shell. (5) 'Generate Infracost Diff': --project-name "${{ inputs.stack }}-${{ inputs.component }}". (6) 'Store Component Metadata': echo with ${{ inputs.stack }}, ${{ inputs.component }} directly. (7) 'Publish Summary': if [[ "${{ inputs.drift-detection-mode-enabled }}" == "true" ]], STEP_SUMMARY_FILE="${{ steps.vars.outputs.issue_file }}".

Locations:

- `action.yml:91`
- `action.yml:247`
- `action.yml:302`
- `action.yml:395`
- `action.yml:415`
- `action.yml:636`
- `action.yml:680`
- `action.yml:720`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs or step outputs to $GITHUB_ENV and $GITHUB_OUTPUT without the required sanitization (printf '%s' ... | tr -d '\n\r'), enabling newline injection. (1) Step 'Set atmos cli config path vars': echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV — inputs.atmos-config-path written to $GITHUB_ENV without sanitization. (2) Step 'Set atmos cli base path vars': ATMOS_BASE_PATH set from step output then echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV — step output written to $GITHUB_ENV without sanitization. (3) Step 'Define Job Variables': echo "stack_name=${STACK_NAME}" >> $GITHUB_OUTPUT, echo "component_name=${COMPONENT_NAME}" >> $GITHUB_OUTPUT, echo "plan_file=${PLAN_FILE}" >> $GITHUB_OUTPUT, etc. — values derived from inputs.stack, inputs.component, inputs.sha, and step outputs written to $GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:91`
- `action.yml:247`
- `action.yml:316`
- `action.yml:330`
- `action.yml:335`

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

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection

**Notes:**

Rewrote hardened/action/action.yml to fix all security findings:

1. unpinned-uses: Pinned all 10 unpinned action references to full 40-char SHA digests (actions/checkout, cloudposse/github-action-setup-atmos, cloudposse/github-action-atmos-get-setting, aws-actions/configure-aws-credentials x2, actions/cache, cloudposse/github-action-terraform-plan-storage x2, infracost/actions/setup, actions/upload-artifact). The two already-pinned actions were preserved.

2. script-injection / static-inline-injection: Moved all ${{ inputs.* }}, ${{ github.* }}, ${{ steps.*.outputs.* }}, and ${{ fromJson(...) }} expressions out of run: blocks into env: blocks for every affected step (Set atmos cli config path vars, Add Terraform and OpenTofu to Aqua, Set atmos cli base path vars, Define Job Variables, Atmos Terraform Plan, Set Plan Results, Generate Infracost Diff, Debug Infracost, Set Infracost Variables, Store Component Metadata to Artifacts, Publish Summary, Exit status).

3. github-env-injection: Added sanitization using printf '%s' ... | tr -d '\n\r' before writing values to $GITHUB_ENV (in Set atmos cli config path vars and Set atmos cli base path vars) and $GITHUB_OUTPUT (in Define Job Variables for all outputs derived from inputs).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in action.yml:

1. 'Add Terraform and OpenTofu to Aqua' step (lines ~175, ~184): Changed `$VERSION` to `${VERSION}` in echo commands. The variable was already inside double quotes so was protected from word-splitting, but the explicit braces make the boundary clear.

2. 'Atmos Terraform Plan' step (lines ~310, ~330):
   - Converted `base_cmd` from a string to a bash array (`base_cmd=()`), with elements added via `base_cmd+=("--identity=$INPUT_IDENTITY")` and expanded as `"${base_cmd[@]}"`. This prevents word-splitting and glob expansion on the identity value.
   - Converted `atmos_pro_flags` from a string to a bash array similarly.
   - Replaced the command substitution `$([[ "$INPUT_PR_COMMENT" == "false" ]] && echo "--output $VARS_SUMMARY_FILE")` with a `tfcmt_output_flags` bash array that is conditionally populated with `("--output" "$VARS_SUMMARY_FILE")` and expanded as `"${tfcmt_output_flags[@]}"`. This properly quotes `$VARS_SUMMARY_FILE` and prevents word-splitting/glob expansion.
   - Replaced the `$([[ "$INPUT_PR_COMMENT" == "true" ]] && echo "-patch")` command substitution with a `tfcmt_patch_flags` array for consistency.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerability in action.yml at line 399. The `$ATMOS_COMMAND` variable (set from `steps.atmos-settings.outputs.settings.command`, a workflow-controllable step output) was used unquoted as a shell command: `$ATMOS_COMMAND show -json "$VARS_PLAN_FILE"`. Changed to `"$ATMOS_COMMAND" show -json "$VARS_PLAN_FILE"` to prevent shell metacharacter injection.

