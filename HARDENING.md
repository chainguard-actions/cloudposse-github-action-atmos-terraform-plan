<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-plan/v5.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-plan/v5.2.1** was hardened automatically. 38 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten. Unpinned references: `actions/checkout@v4`, `cloudposse/github-action-setup-atmos@v2`, `cloudposse/github-action-atmos-get-setting@v2`, `aws-actions/configure-aws-credentials@v4` (used twice), `actions/cache@v4`, `cloudposse/github-action-terraform-plan-storage@v1` (used twice), `infracost/actions/setup@v3`, `actions/upload-artifact@v4`. (Note: `actions/cache@5a3ec84eff668545956fd18022155c47e93e2684` and `aquaproj/aqua-installer@5e54e5cee8a95ee2ce7c04cb993da6dfad13e59c` are correctly pinned.)

Locations:

- `action.yml:83`
- `action.yml:87`
- `action.yml:93`
- `action.yml:196`
- `action.yml:213`
- `action.yml:247`
- `action.yml:261`
- `action.yml:278`
- `action.yml:295`
- `action.yml:380`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml interpolate `${{ ... }}` expressions directly into shell commands (sub-rule a), allowing an attacker-controlled value to break out of the intended shell context. Affected steps and representative offending lines:

1. **Set atmos cli config path vars** — `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — `inputs.atmos-config-path` is interpolated directly into a shell command.

2. **Set atmos cli base path vars** — `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` — step output interpolated directly.

3. **Define Job Variables** — `STACK_NAME=$(echo "${{ inputs.stack }}" | sed ...)`, `COMPONENT_PATH=${{ fromJson(steps.atmos-settings.outputs.settings).component-path }}` (unquoted), `COMPONENT_NAME=$(echo "${{ inputs.component }}" | sed ...)`, `PLAN_FILE=.../$COMPONENT_SLUG-${{ inputs.sha }}.planfile` — multiple inputs interpolated directly.

4. **Atmos Terraform Plan** — `rm -f ./${{ steps.vars.outputs.component_path }}/.terraform/environment`, `-owner "${{ github.repository_owner }}"`, `-repo "${{ github.event.repository.name }}"`, `-var "component:${{ inputs.component }}"`, `-var "stack:${{ inputs.stack }}"`, `-var "commitSHA:${{ inputs.sha }}"`, `atmos terraform plan ${{ inputs.component }}`, `--stack ${{ inputs.stack }}`, `-out="${{ steps.vars.outputs.plan_file }}"`, `${{ fromJson(steps.atmos-settings.outputs.settings).command }} show -json ...` — numerous expressions interpolated directly.

5. **Set Plan Results** — `echo "plan_file=${{ steps.vars.outputs.plan_file }}" >> $GITHUB_OUTPUT` — step output interpolated directly.

6. **Generate Infracost Diff** — `--path="${{ steps.vars.outputs.plan_file }}.json"`, `--project-name "${{ inputs.stack }}-${{ inputs.component }}"` — inputs interpolated directly.

7. **Debug Infracost** — `cat ${{ steps.vars.outputs.plan_file }}.json` — step output interpolated directly.

8. **Set Infracost Variables** — `sed -i ... ${{ steps.vars.outputs.step_summary_file }}` — step output interpolated directly.

9. **Store Component Metadata** — `echo -n '{ "stack": "${{ inputs.stack }}", "component": "${{ inputs.component }}", ... }' > "metadata/${{ steps.vars.outputs.component_slug }}.metadata.json"` — inputs interpolated directly.

10. **Publish Summary** — `STEP_SUMMARY_FILE="${{ steps.vars.outputs.issue_file }}"`, `if [[ "${{ inputs.drift-detection-mode-enabled }}" == "true" ]]` — inputs and step outputs interpolated directly.

Locations:

- `action.yml:88`
- `action.yml:204`
- `action.yml:218`
- `action.yml:222`
- `action.yml:224`
- `action.yml:226`
- `action.yml:240`
- `action.yml:248`
- `action.yml:302`
- `action.yml:316`
- `action.yml:330`
- `action.yml:340`
- `action.yml:355`
- `action.yml:362`
- `action.yml:370`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs to `$GITHUB_ENV` or `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`), enabling newline injection that can set arbitrary environment variables or outputs.

1. **Set atmos cli config path vars** (line ~88): `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — `inputs.atmos-config-path` (caller-controlled) is written directly to `$GITHUB_ENV` without sanitization.

2. **Set atmos cli base path vars** (line ~204): `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` then `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV` — a step output (ultimately derived from caller-controlled atmos config) is written to `$GITHUB_ENV` without sanitization.

3. **Define Job Variables** (line ~218): Values derived from `${{ inputs.stack }}`, `${{ inputs.component }}`, and `${{ inputs.sha }}` are written to `$GITHUB_OUTPUT` (e.g., `echo "stack_name=${STACK_NAME}" >> $GITHUB_OUTPUT`) without sanitization.

4. **Set Plan Results** (line ~302): `echo "plan_file=${{ steps.vars.outputs.plan_file }}" >> $GITHUB_OUTPUT` — a step output (path derived from inputs) is written to `$GITHUB_OUTPUT` without sanitization.

5. **Publish Summary** (line ~355): `echo "result=${STEP_SUMMARY}" >> $GITHUB_OUTPUT` — file content (derived from terraform plan output, which may contain attacker-influenced data) is written to `$GITHUB_OUTPUT` without sanitization.

Locations:

- `action.yml:88`
- `action.yml:204`
- `action.yml:218`
- `action.yml:302`
- `action.yml:355`

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

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all security findings in hardened/action/action.yml:

1. **unpinned-uses**: Pinned all 10 unpinned `uses:` references to full 40-character SHA digests using lookup_action_sha. Preserved mutable tag names as inline comments.

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions out of `run:` blocks into `env:` maps for every affected step (Set atmos cli config path vars, Add Terraform and OpenTofu to Aqua, Set atmos cli base path vars, Define Job Variables, Atmos Terraform Plan, Set Plan Results, Generate Infracost Diff, Debug Infracost, Set Infracost Variables, Store Component Metadata to Artifacts, Publish Summary, Exit status). Shell scripts now reference plain environment variables.

3. **github-env-injection**: All values derived from inputs or step outputs that are written to $GITHUB_ENV or $GITHUB_OUTPUT are now sanitized with `printf '%s' ... | tr -d '\n\r'` before writing, preventing newline injection.

The already-pinned references (actions/cache@5a3ec84... and aquaproj/aqua-installer@5e54e5c...) were preserved as-is.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings in hardened/action/action.yml:
1. Lines 213-215, 221 ('Define Job Variables' step): Added double-quotes around ${COMPONENT_PATH} in all four realpath calls: `realpath "${COMPONENT_PATH}"` and `realpath --relative-to="${PWD}" "${COMPONENT_PATH}"`.
2. Line 299 ('Atmos Terraform Plan' step): Replaced the unquoted command substitution `$([[ "$INPUT_PR_COMMENT" == "false" ]] && echo "--output ${VARS_SUMMARY_FILE}")` with a bash array approach: conditionally assign `tfcmt_output_args=(--output "${VARS_SUMMARY_FILE}")` then expand as `"${tfcmt_output_args[@]}"`, properly quoting the path.
3. Lines 497, 501, 505 ('Publish Summary or Generate GitHub Issue Description for Drift Detection' step): Added double-quotes around ${STEP_SUMMARY_FILE} in `rm -f "${STEP_SUMMARY_FILE}"`, `if [ -f "${STEP_SUMMARY_FILE}" ]`, and `cat "${STEP_SUMMARY_FILE}"`.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Set atmos cli config path vars' step in action.yml (line 82). The `realpath` output was being written directly to $GITHUB_ENV without sanitization. Changed the code to capture the realpath output into a variable `safe_config_path` using `printf '%s' "$(realpath "$safe_path")" | tr -d '\n\r'`, then write `${safe_config_path}` to $GITHUB_ENV. This matches the correct pattern already used in the adjacent 'Set atmos cli base path vars' step.

