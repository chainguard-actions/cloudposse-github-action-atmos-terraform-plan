<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-plan/v5.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-plan/v5.2.1** was hardened automatically. 38 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable version tags instead of full 40-character SHA commit digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten. Unpinned references: `actions/checkout@v4`, `cloudposse/github-action-setup-atmos@v2`, `cloudposse/github-action-atmos-get-setting@v2`, `aws-actions/configure-aws-credentials@v4` (×2), `actions/cache@v4`, `cloudposse/github-action-terraform-plan-storage@v1` (×2), `infracost/actions/setup@v3`, `actions/upload-artifact@v4`.

Locations:

- `action.yml:82`
- `action.yml:89`
- `action.yml:95`
- `action.yml:218`
- `action.yml:233`
- `action.yml:258`
- `action.yml:390`
- `action.yml:416`
- `action.yml:444`
- `action.yml:530`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ inputs.* }}`, `${{ github.* }}`, and `${{ steps.*.outputs.* }}` expressions into shell commands (rule a). This allows an attacker who controls input values (e.g. via a malicious PR or workflow_dispatch) to inject arbitrary shell commands. Affected steps and offending expressions include:

1. **Set atmos cli config path vars**: `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — `inputs.atmos-config-path` interpolated directly.

2. **Add Terraform and OpenTofu to Aqua**: `${{ fromJson(steps.atmos-settings.outputs.settings).terraform-version }}`, `${{ fromJson(steps.atmos-settings.outputs.settings).opentofu-version }}`, and `${{ inputs.debug }}` interpolated directly into shell conditionals and echo commands.

3. **Set atmos cli base path vars**: `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` — steps output interpolated directly.

4. **Define Job Variables**: `${{ inputs.stack }}`, `${{ inputs.component }}`, `${{ inputs.sha }}`, and `${{ fromJson(steps.atmos-settings.outputs.settings).component-path }}` interpolated directly into shell variable assignments and path constructions.

5. **Atmos Terraform Plan**: `${{ inputs.atmos-pro-upload-deployment-status }}`, `${{ github.repository_owner }}`, `${{ github.event.repository.name }}`, `${{ inputs.component }}`, `${{ inputs.stack }}`, `${{ inputs.sha }}`, `${{ inputs.drift-detection-mode-enabled }}`, `${{ inputs.pr-comment }}`, `${{ inputs.debug }}`, `${{ fromJson(...).command }}` all interpolated directly into shell command arguments.

6. **Set Plan Results**: `${{ steps.vars.outputs.plan_file }}` and `${{ steps.vars.outputs.plan_file_json }}` interpolated directly.

7. **Generate Infracost Diff**: `${{ inputs.stack }}` and `${{ inputs.component }}` interpolated directly into CLI arguments.

8. **Store Component Metadata to Artifacts**: `${{ inputs.stack }}`, `${{ inputs.component }}`, `${{ steps.vars.outputs.component_path }}` interpolated directly.

9. **Publish Summary**: `${{ inputs.drift-detection-mode-enabled }}` and `${{ steps.atmos-plan.outputs.no-changes }}` interpolated directly.

10. **Exit status**: `exit ${{ steps.atmos-plan.outputs.result }}` — steps output interpolated directly into exit command.

Locations:

- `action.yml:87`
- `action.yml:107`
- `action.yml:176`
- `action.yml:192`
- `action.yml:207`
- `action.yml:270`
- `action.yml:300`
- `action.yml:460`
- `action.yml:490`
- `action.yml:510`
- `action.yml:540`
- `action.yml:560`

### github-env-injection (severity: high)

Multiple `run:` blocks write untrusted input values to `$GITHUB_ENV` and `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`), enabling newline-injection attacks that can set arbitrary environment variables or outputs:

1. **Set atmos cli config path vars** (writes to `$GITHUB_ENV`): `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — `inputs.atmos-config-path` is caller-controlled and written directly to GITHUB_ENV without sanitization.

2. **Set atmos cli base path vars** (writes to `$GITHUB_ENV`): `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` then `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV` — the `steps.*.outputs.*` value is untrusted and written to GITHUB_ENV without sanitization.

3. **Define Job Variables** (writes to `$GITHUB_OUTPUT`): `STACK_NAME=$(echo "${{ inputs.stack }}" | sed 's#/#_#g')` and `COMPONENT_NAME=$(echo "${{ inputs.component }}" | sed 's#/#_#g')` are derived from caller-controlled inputs and then written via `echo "stack_name=${STACK_NAME}" >> $GITHUB_OUTPUT` etc., without newline sanitization. The `sed` substitution only replaces `/` with `_` and does not strip newlines.

Locations:

- `action.yml:87`
- `action.yml:176`
- `action.yml:207`
- `action.yml:213`

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

Rewrote action.yml to fix all security findings:

1. Pinned all 10 unpinned action references to full 40-character SHA digests (actions/checkout, cloudposse/github-action-setup-atmos, cloudposse/github-action-atmos-get-setting, aws-actions/configure-aws-credentials ×2, actions/cache, cloudposse/github-action-terraform-plan-storage ×2, infracost/actions/setup, actions/upload-artifact).

2. Moved all ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions from run: shell blocks into step env: blocks, referencing them as plain environment variables in the shell scripts. Affected steps: Set atmos cli config path vars, Add Terraform and OpenTofu to Aqua, Set atmos cli base path vars, Define Job Variables, Atmos Terraform Plan, Set Plan Results, Generate Infracost Diff, Debug Infracost, Set Infracost Variables, Store Component Metadata to Artifacts, Publish Summary, Exit status.

3. Added newline sanitization (printf '%s' ... | tr -d '\n\r') for all values written to GITHUB_ENV (atmos-config-path in Set atmos cli config path vars, base-path in Set atmos cli base path vars) and for inputs.stack and inputs.component before they are used to construct paths/filenames in Define Job Variables.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all four security findings in hardened/action/action.yml:

1. script-injection (Define Job Variables, line 207): Sanitized ATMOS_COMPONENT_PATH with `printf '%s' ... | tr -d '\n\r'` before assigning to COMPONENT_PATH, and added double-quotes around all `realpath "${COMPONENT_PATH}"` calls (three realpath calls for PLAN_FILE, PLAN_FILE_JSON, LOCK_FILE, and one for COMPONENT_RELPATH).

2. script-injection (Atmos Terraform Plan, line 262): The `rm -f` command already had the variable inside a double-quoted string (`"./${VARS_COMPONENT_PATH}/.terraform/environment"`), which is safe from shell metacharacter injection.

3. github-env-injection (Define Job Variables, line 218): Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization for all values derived from COMPONENT_PATH (component_path, cache-key, plan_file, plan_file_json, lock_file, summary_file, step_summary_file, issue_file) before writing to $GITHUB_OUTPUT.

4. github-env-injection (Set Plan Results, line 421): Added `printf '%s' "$VARS_PLAN_FILE" | tr -d '\n\r'` and `printf '%s' "$VARS_PLAN_FILE_JSON" | tr -d '\n\r'` sanitization before writing plan_file and plan_file_json to $GITHUB_OUTPUT.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted `$ATMOS_COMMAND` variable expansion in the 'Atmos Terraform Plan' run block at action.yml line ~430. Changed `$ATMOS_COMMAND show -json "$VARS_PLAN_FILE"` to `"$ATMOS_COMMAND" show -json "$VARS_PLAN_FILE"`. Double-quoting the variable prevents shell word-splitting and glob expansion, which could allow an attacker who controls the atmos settings output to inject shell metacharacters and execute arbitrary commands.

