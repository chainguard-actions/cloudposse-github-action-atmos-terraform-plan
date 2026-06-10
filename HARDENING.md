<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-plan/v5.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **cloudposse--github-action-atmos-terraform-plan/v5.2.1** was hardened automatically. 38 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions (rule a), allowing script injection. Affected steps and examples:

1. **Set atmos cli config path vars** (~line 97): `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — `inputs.atmos-config-path` is interpolated directly into the shell command.

2. **Set atmos cli base path vars** (~line 219): `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` — step output interpolated directly.

3. **Add Terraform and OpenTofu to Aqua** (~line 172): `if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).terraform-version }}" != "" ...` and `VERSION="${{ fromJson(steps.atmos-settings.outputs.settings).terraform-version }}"` — step outputs interpolated directly.

4. **Define Job Variables** (~line 237): `STACK_NAME=$(echo "${{ inputs.stack }}" | sed ...)`, `COMPONENT_PATH=${{ fromJson(steps.atmos-settings.outputs.settings).component-path }}`, `COMPONENT_NAME=$(echo "${{ inputs.component }}" | sed ...)`, `PLAN_FILE="...${{ inputs.sha }}.planfile"` — multiple inputs interpolated directly.

5. **Atmos Terraform Plan** (~line 283): `-owner "${{ github.repository_owner }}"`, `-repo "${{ github.event.repository.name }}"`, `-var "component:${{ inputs.component }}"`, `-var "stack:${{ inputs.stack }}"`, `atmos terraform plan ${{ inputs.component }} --stack ${{ inputs.stack }}`, `${{ fromJson(steps.atmos-settings.outputs.settings).command }} show -json ...` — many github/inputs/steps contexts interpolated directly.

6. **Set Plan Results** (~line 480): `if [[ -f "${{ steps.vars.outputs.plan_file }}" ]]` and `echo "plan_file=${{ steps.vars.outputs.plan_file }}" >> $GITHUB_OUTPUT`.

7. **Generate Infracost Diff** (~line 497): `--path="${{ steps.vars.outputs.plan_file }}.json"`, `--project-name "${{ inputs.stack }}-${{ inputs.component }}"`.

8. **Debug Infracost** (~line 511): `cat ${{ steps.vars.outputs.plan_file }}.json`.

9. **Set Infracost Variables** (~line 519): `sed -i "..." ${{ steps.vars.outputs.step_summary_file }}`.

10. **Store Component Metadata to Artifacts** (~line 540): `echo -n '{ "stack": "${{ inputs.stack }}", "component": "${{ inputs.component }}" ... }'`.

11. **Publish Summary** (~line 553): `if [[ "${{ inputs.drift-detection-mode-enabled }}" == "true" ]]`, `if [[ "${{ steps.atmos-plan.outputs.no-changes }}" == "true" ]]`.

Locations:

- `action.yml:97`
- `action.yml:219`
- `action.yml:172`
- `action.yml:237`
- `action.yml:283`
- `action.yml:480`
- `action.yml:497`
- `action.yml:511`
- `action.yml:519`
- `action.yml:540`
- `action.yml:553`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs to `$GITHUB_ENV` or `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. **Set atmos cli config path vars** (~line 97): `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — `inputs.atmos-config-path` (caller-controlled) is written directly to `$GITHUB_ENV` with no newline sanitization.

2. **Set atmos cli base path vars** (~line 220): `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` then `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV` — a step output (which is ultimately derived from caller-controlled atmos config) is written to `$GITHUB_ENV` without sanitization.

3. **Define Job Variables** (~line 248–258): `STACK_NAME` (derived from `${{ inputs.stack }}`), `COMPONENT_NAME` (derived from `${{ inputs.component }}`), `PLAN_FILE` (derived from `${{ inputs.sha }}`), and other variables derived from step outputs are all written to `$GITHUB_OUTPUT` via `echo "...=${VAR}" >> $GITHUB_OUTPUT` without sanitization. A newline in any of these inputs could inject arbitrary environment variables or outputs.

Locations:

- `action.yml:97`
- `action.yml:220`
- `action.yml:248`

### unpinned-uses (severity: high)

The following `uses:` references in action.yml use mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised:

- `uses: actions/checkout@v4` (Checkout step)
- `uses: cloudposse/github-action-setup-atmos@v2` (Install Atmos step)
- `uses: cloudposse/github-action-atmos-get-setting@v2` (Get atmos settings step)
- `uses: aws-actions/configure-aws-credentials@v4` (Configure Plan AWS Credentials step)
- `uses: actions/cache@v4` (Cache .terraform step)
- `uses: aws-actions/configure-aws-credentials@v4` (Configure State AWS Credentials step)
- `uses: cloudposse/github-action-terraform-plan-storage@v1` (Store New Plan step)
- `uses: cloudposse/github-action-terraform-plan-storage@v1` (Store Lockfile for New Plan step)
- `uses: infracost/actions/setup@v3` (Setup Infracost step)
- `uses: actions/upload-artifact@v4` (Upload Artifacts step)

Note: `actions/cache@5a3ec84eff668545956fd18022155c47e93e2684` and `aquaproj/aqua-installer@5e54e5cee8a95ee2ce7c04cb993da6dfad13e59c` are correctly pinned to SHAs.

Locations:

- `action.yml:90`
- `action.yml:101`
- `action.yml:106`
- `action.yml:210`
- `action.yml:278`
- `action.yml:404`
- `action.yml:421`
- `action.yml:443`
- `action.yml:476`
- `action.yml:571`

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

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed all security findings in action.yml:

1. **unpinned-uses**: Pinned all 10 action references to full 40-character commit SHAs (actions/checkout, cloudposse/github-action-setup-atmos, cloudposse/github-action-atmos-get-setting, aws-actions/configure-aws-credentials x2, actions/cache, cloudposse/github-action-terraform-plan-storage x2, infracost/actions/setup, actions/upload-artifact).

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions out of run: blocks and into env: blocks across 11 affected steps. Shell scripts now reference plain environment variables ($VAR_NAME) instead of inline GitHub expressions.

3. **github-env-injection**: Added `printf '%s' ... | tr -d '\n\r'` sanitization for all values derived from user-controlled inputs before writing to $GITHUB_ENV and $GITHUB_OUTPUT. This prevents newline injection attacks in the Set atmos cli config path vars, Set atmos cli base path vars, Define Job Variables, and Set Plan Results steps.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in action.yml:

1. Line ~430: Quoted `$ATMOS_COMMAND` → `"$ATMOS_COMMAND"` to prevent word-splitting and metacharacter injection when the variable (derived from workflow-controllable step output) is used as a shell command.

2. Line ~370: Replaced the unquoted inline command substitution `$([[ "$INPUT_PR_COMMENT" == "false" ]] && echo "--output $VARS_SUMMARY_FILE")` with a pre-computed bash array approach. Now `tfcmt_output_flags`, `tfcmt_patch_flags`, and `tfcmt_log_level` are computed before the tfcmt call using proper if-statements, and the arrays are expanded as `"${tfcmt_output_flags[@]}"` and `"${tfcmt_patch_flags[@]}"` to prevent word-splitting and glob expansion of attacker-controlled path values. Also updated the second tfcmt call to use the pre-computed `$tfcmt_log_level` variable.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion of `${COMPONENT_PATH}` in the 'Define Job Variables' step of action.yml. Added double quotes around `${COMPONENT_PATH}` in all four `realpath` calls: PLAN_FILE, PLAN_FILE_JSON, LOCK_FILE assignments, and the COMPONENT_RELPATH assignment. This prevents shell metacharacter injection from attacker-controlled `component-path` atmos settings.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted expansions of `${STEP_SUMMARY_FILE}` / `$STEP_SUMMARY_FILE` in the 'Publish Summary or Generate GitHub Issue Description for Drift Detection' step of action.yml. Changed: (1) `rm -f ${STEP_SUMMARY_FILE}` → `rm -f "${STEP_SUMMARY_FILE}"`, (2) `if [ -f ${STEP_SUMMARY_FILE} ]` → `if [ -f "${STEP_SUMMARY_FILE}" ]`, (3) `cat $STEP_SUMMARY_FILE >> $GITHUB_STEP_SUMMARY` → `cat "$STEP_SUMMARY_FILE" >> $GITHUB_STEP_SUMMARY`, and also fixed the additional unquoted `cat ${STEP_SUMMARY_FILE}` in the same block. All occurrences are now properly double-quoted to prevent word splitting and glob expansion.

