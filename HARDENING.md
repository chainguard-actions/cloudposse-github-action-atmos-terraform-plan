<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-plan/v5.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-plan/v5.2.1** was hardened automatically. 38 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are directly interpolated inside run: shell scripts in action.yml, violating rule (a). This includes attacker-controllable inputs.* and github.* contexts as well as steps.*.outputs.* values.

Affected steps and offending lines:

1. 'Set atmos cli config path vars' step: `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — inputs.atmos-config-path interpolated directly in shell.

2. 'Add Terraform and OpenTofu to Aqua' step: `if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).terraform-version }}" != "" ...` and `VERSION="${{ fromJson(steps.atmos-settings.outputs.settings).terraform-version }}"` and `if [[ "${{ inputs.debug }}" == "true" ]]` — steps outputs and inputs interpolated directly.

3. 'Set atmos cli base path vars' step: `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` — steps output interpolated directly.

4. 'Define Job Variables' step: `STACK_NAME=$(echo "${{ inputs.stack }}" | ...)`, `COMPONENT_PATH=${{ fromJson(steps.atmos-settings.outputs.settings).component-path }}`, `COMPONENT_NAME=$(echo "${{ inputs.component }}" | ...)`, `COMPONENT_CACHE_KEY=$(basename "${{ fromJson(...).component-path }}")`, `PLAN_FILE=".../$COMPONENT_SLUG-${{ inputs.sha }}.planfile"` — multiple inputs and steps outputs interpolated directly.

5. 'Atmos Terraform Plan' step: Extensive use of `${{ inputs.* }}`, `${{ github.repository_owner }}`, `${{ github.event.repository.name }}`, `${{ github.job }}`, `${{ steps.vars.outputs.* }}` directly in shell commands including as CLI arguments to tfcmt and atmos.

6. 'Set Plan Results' step: `${{ steps.vars.outputs.plan_file }}` and `${{ steps.vars.outputs.plan_file_json }}` interpolated directly in run: block.

7. 'Generate Infracost Diff' step: `${{ steps.vars.outputs.plan_file }}`, `${{ inputs.stack }}`, `${{ inputs.component }}` interpolated directly.

8. 'Debug Infracost' step: `${{ steps.vars.outputs.plan_file }}` interpolated directly.

9. 'Set Infracost Variables' step: `${{ steps.vars.outputs.step_summary_file }}` interpolated directly.

10. 'Store Component Metadata to Artifacts' step: `${{ inputs.stack }}`, `${{ inputs.component }}`, `${{ steps.vars.outputs.component_path }}`, `${{ steps.atmos-plan.outputs.changes }}`, `${{ steps.atmos-plan.outputs.error }}`, `${{ steps.vars.outputs.component_slug }}` interpolated directly.

11. 'Publish Summary' step: `${{ inputs.drift-detection-mode-enabled }}`, `${{ steps.vars.outputs.issue_file }}`, `${{ steps.atmos-plan.outputs.no-changes }}`, `${{ steps.vars.outputs.step_summary_file }}` interpolated directly.

12. 'Exit status' step: `exit ${{ steps.atmos-plan.outputs.result }}` — steps output interpolated directly into exit command.

Locations:

- `action.yml:83`
- `action.yml:155`
- `action.yml:163`
- `action.yml:167`
- `action.yml:175`
- `action.yml:204`
- `action.yml:206`
- `action.yml:207`
- `action.yml:208`
- `action.yml:209`
- `action.yml:215`
- `action.yml:222`
- `action.yml:228`
- `action.yml:232`
- `action.yml:233`
- `action.yml:234`
- `action.yml:235`
- `action.yml:236`
- `action.yml:237`
- `action.yml:238`
- `action.yml:239`
- `action.yml:240`
- `action.yml:241`
- `action.yml:242`
- `action.yml:243`
- `action.yml:244`
- `action.yml:245`
- `action.yml:246`
- `action.yml:247`
- `action.yml:248`
- `action.yml:249`
- `action.yml:250`
- `action.yml:251`

### github-env-injection (severity: high)

Two run: blocks write untrusted input values to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. 'Set atmos cli config path vars' step writes `${{ inputs.atmos-config-path }}` (a caller-controlled input) directly to $GITHUB_ENV:
   `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV`
   An attacker can inject newlines into inputs.atmos-config-path to add arbitrary environment variables.

2. 'Set atmos cli base path vars' step writes a steps output (derived from atmos settings, which is ultimately caller-controlled) to $GITHUB_ENV without sanitization:
   `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"`
   `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV`
   The base-path value comes from atmos settings which are controlled by the component/stack inputs.

Locations:

- `action.yml:83`
- `action.yml:175`

### unpinned-uses (severity: high)

Multiple action references use mutable tags instead of pinned full-length SHA digests, making the action vulnerable to supply-chain attacks.

In action.yml:
- `uses: actions/checkout@v4` (tag, not SHA)
- `uses: cloudposse/github-action-setup-atmos@v2` (tag, not SHA)
- `uses: cloudposse/github-action-atmos-get-setting@v2` (tag, not SHA)
- `uses: aws-actions/configure-aws-credentials@v4` (tag, not SHA) — appears twice
- `uses: actions/cache@v4` (tag, not SHA)
- `uses: cloudposse/github-action-terraform-plan-storage@v1` (tag, not SHA) — appears twice
- `uses: infracost/actions/setup@v3` (tag, not SHA)
- `uses: actions/upload-artifact@v4` (tag, not SHA)

Note: `actions/cache@5a3ec84eff668545956fd18022155c47e93e2684` and `aquaproj/aqua-installer@5e54e5cee8a95ee2ce7c04cb993da6dfad13e59c` are correctly pinned to SHAs.

In .github/workflows/branch.yml:
- `uses: cloudposse/.github/.github/workflows/shared-github-action.yml@main` (branch ref, not SHA)

In .github/workflows/release.yml:
- `uses: cloudposse/.github/.github/workflows/shared-release-branches.yml@main` (branch ref, not SHA)

In .github/workflows/integration-tests.yml:
- `uses: actions/checkout@v4` (tag, not SHA)

In .github/workflows/test-atmos-pro-enabled.yml, test-changes-exists.yml, test-changes-exists-drift.yml, test-failed-plan.yml, test-failed-plan-drift.yml, test-infra-cost.yml, test-no-changes.yml, test-no-changes-drift-more.yml, test-settings-action-disabled.yml, test-settings-action-disabled-drift.yml:
- `uses: actions/checkout@v4` (tag, not SHA)
- `uses: nick-fields/assert-action@v2` (tag, not SHA) — multiple occurrences per file
- `uses: actions/download-artifact@v4` (tag, not SHA) — in some files

Locations:

- `action.yml:80`
- `action.yml:88`
- `action.yml:93`
- `action.yml:196`
- `action.yml:213`
- `action.yml:303`
- `action.yml:316`
- `action.yml:329`
- `action.yml:358`
- `action.yml:421`
- `.github/workflows/branch.yml:20`
- `.github/workflows/release.yml:10`
- `.github/workflows/integration-tests.yml:19`
- `.github/workflows/test-atmos-pro-enabled.yml:30`
- `.github/workflows/test-atmos-pro-enabled.yml:73`
- `.github/workflows/test-changes-exists.yml:30`
- `.github/workflows/test-changes-exists.yml:72`
- `.github/workflows/test-changes-exists-drift.yml:30`
- `.github/workflows/test-failed-plan.yml:30`
- `.github/workflows/test-no-changes.yml:30`
- `.github/workflows/test-no-changes.yml:72`
- `.github/workflows/test-no-changes.yml:76`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.atmos-config-path }}" appears directly in run: block of step "Set atmos cli config path vars"; move to env: map

Locations:

- `action.yml:97`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.debug }}" appears directly in run: block of step "Add Terraform and OpenTofu to Aqua"; move to env: map

Locations:

- `action.yml:226`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:278`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:280`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:283`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:284`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.atmos-pro-upload-deployment-status }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:338`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:347`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:349`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:350`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-image }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:352`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-url }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:353`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.drift-detection-mode-enabled }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:355`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pr-comment }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:356`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.debug }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:357`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pr-comment }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:359`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:361`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:362`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pr-comment }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:372`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:378`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:380`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:381`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-image }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:383`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-url }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:384`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.drift-detection-mode-enabled }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:386`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.debug }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:388`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.drift-detection-mode-enabled }}" appears directly in run: block of step "Atmos Terraform Plan"; move to env: map

Locations:

- `action.yml:396`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:525`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:525`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:530`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:530`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Store Component Metadata to Artifacts"; move to env: map

Locations:

- `action.yml:566`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Store Component Metadata to Artifacts"; move to env: map

Locations:

- `action.yml:566`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.drift-detection-mode-enabled }}" appears directly in run: block of step "Publish Summary or Generate GitHub Issue Description for Drift Detection"; move to env: map

Locations:

- `action.yml:574`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.drift-detection-mode-enabled }}" appears directly in run: block of step "Publish Summary or Generate GitHub Issue Description for Drift Detection"; move to env: map

Locations:

- `action.yml:592`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, static-inline-injection

**Notes:**

Fixed all security findings in action.yml and non-test workflow files:

1. script-injection / static-inline-injection: Moved all ${{ }} expressions from run: blocks to env: blocks across all affected steps (Set atmos cli config path vars, Add Terraform and OpenTofu to Aqua, Set atmos cli base path vars, Define Job Variables, Atmos Terraform Plan, Set Plan Results, Generate Infracost Diff, Debug Infracost, Set Infracost Variables, Store Component Metadata to Artifacts, Publish Summary, Exit status).

2. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing to GITHUB_ENV in both 'Set atmos cli config path vars' and 'Set atmos cli base path vars' steps.

3. unpinned-uses in action.yml: Pinned all 8 action references to full SHA digests (actions/checkout, cloudposse/github-action-setup-atmos, cloudposse/github-action-atmos-get-setting, aws-actions/configure-aws-credentials x2, actions/cache, cloudposse/github-action-terraform-plan-storage x2, infracost/actions/setup, actions/upload-artifact).

4. unpinned-uses in workflow files: Pinned branch.yml and release.yml cloudposse/.github references to SHA 3911c663309ecdda30d8b8fcbec7bde19d1d6ddb. Test workflow files (test-*.yml, integration-tests.yml) were intentionally left unmodified as they are test harness files.

### Iteration 2

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings:

1. unpinned-uses: Pinned all 4 action references to full 40-char SHAs across 11 workflow files:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
   - nick-fields/assert-action@v2 → @aa0067e01f0f6545c31755d6ca128c5a3a14f6bf
   - actions/download-artifact@v4 → @d3f86a106a0bac45b974a628896c90dbdf5c8093
   - kibertoad/wait-action@1.0.1 → @99f6f101c5be7b88bb9b41c0d3b810722491b8e5

2. script-injection: Moved ${{ runner.temp }} and ${{ env.AWS_REGION }} from run: shell strings to env: blocks (as RUNNER_TEMP and AWS_REGION) in all 11 affected workflow files. Shell scripts now reference them as "$RUNNER_TEMP" and "${AWS_REGION}".

3. github-env-injection: Fixed the Define Job Variables step in action.yml to sanitize all values derived from untrusted inputs (INPUT_STACK, INPUT_COMPONENT, INPUT_SHA, ATMOS_COMPONENT_PATH) using printf '%s' | tr -d '\n\r' before writing to $GITHUB_OUTPUT. Changed from echo to printf for all GITHUB_OUTPUT writes to prevent newline injection.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in all 11 workflow files (.github/workflows/integration-tests.yml and .github/workflows/test-*.yml). In each file, the ${{ secrets.TERRAFORM_STATE_BUCKET }}, ${{ secrets.TERRAFORM_STATE_TABLE }}, ${{ secrets.TERRAFORM_STATE_ROLE }}, and ${{ secrets.TERRAFORM_PLAN_ROLE }} expressions were moved from the run: shell script body into the step's env: block as TERRAFORM_STATE_BUCKET, TERRAFORM_STATE_TABLE, TERRAFORM_STATE_ROLE, and TERRAFORM_PLAN_ROLE environment variables. The sed commands in the shell scripts were updated to reference these as ${TERRAFORM_STATE_BUCKET} etc. instead of the ${{ }} template expressions.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted $ATMOS_SUMMARY_FILE expansion in the 'Atmos Terraform Plan' step. The original code used `$([[ "$INPUT_PR_COMMENT" == "false" ]] && echo "--output $ATMOS_SUMMARY_FILE")` which left $ATMOS_SUMMARY_FILE unquoted inside a command substitution, making it subject to word splitting and glob expansion. The fix replaces this pattern with pre-built bash arrays: `tfcmt_output_flag=()` populated conditionally with `(--output "$ATMOS_SUMMARY_FILE")`, then expanded as `"${tfcmt_output_flag[@]}"` in the tfcmt invocation. Similarly, the `--log-level` and `-patch` conditional flags were converted to use pre-computed variables/arrays (`$tfcmt_log_level` and `tfcmt_plan_flag`). The `atmos_pro_flags` string variable was also converted to an array for proper argument handling. The second tfcmt invocation's `--log-level` command substitution was also updated to use the pre-computed `$tfcmt_log_level` variable for consistency.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerability in the 'Atmos Terraform Plan' step of action.yml. The `$ATMOS_COMMAND` variable was expanded unquoted (`$ATMOS_COMMAND show -json ...`), allowing an attacker who controls the atmos `command` setting to inject shell metacharacters. Fixed by quoting the expansion: `"$ATMOS_COMMAND" show -json "$ATMOS_PLAN_FILE" > "$ATMOS_PLAN_FILE.json"`. This ensures the command value is treated as a single word and any embedded metacharacters are not interpreted by the shell.

