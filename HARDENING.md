# Hardening Report: cloudposse--github-action-atmos-terraform-plan/v5.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `c40cfe5fa14e08549b1b988e7e5a26da4816abf0`

**Test Policy SHA:** `f2e7d85641cde4267138117189b8eba7ba2bfbde`

Action **cloudposse--github-action-atmos-terraform-plan/v5.2.1** was hardened automatically. 38 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved. Failing references:
- `actions/checkout@v4`
- `cloudposse/github-action-setup-atmos@v2`
- `cloudposse/github-action-atmos-get-setting@v2`
- `aws-actions/configure-aws-credentials@v4` (appears twice)
- `actions/cache@v4`
- `cloudposse/github-action-terraform-plan-storage@v1` (appears twice)
- `infracost/actions/setup@v3`
- `actions/upload-artifact@v4`

Note: `actions/cache@5a3ec84eff668545956fd18022155c47e93e2684` and `aquaproj/aqua-installer@5e54e5cee8a95ee2ce7c04cb993da6dfad13e59c` are correctly pinned.

Locations:

- `action.yml:90`
- `action.yml:100`
- `action.yml:117`
- `action.yml:232`
- `action.yml:296`
- `action.yml:430`
- `action.yml:449`
- `action.yml:472`
- `action.yml:497`
- `action.yml:594`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate GitHub Actions expressions (`${{ inputs.* }}` and `${{ github.* }}`) into shell commands without first assigning them to environment variables. An attacker who controls these values (e.g. via a crafted component/stack name or SHA) could inject arbitrary shell commands.

1. **"Set atmos cli config path vars"** step: `${{ inputs.atmos-config-path }}` is interpolated directly into the shell command.
2. **"Define Job Variables"** step: `${{ inputs.stack }}`, `${{ inputs.component }}`, and `${{ inputs.sha }}` are interpolated directly into shell commands.
3. **"Atmos Terraform Plan"** step: `${{ inputs.component }}`, `${{ inputs.stack }}`, `${{ inputs.sha }}`, `${{ github.repository_owner }}`, and `${{ github.event.repository.name }}` are all interpolated directly into shell commands passed to `tfcmt` and `atmos`.

Locations:

- `action.yml:97`
- `action.yml:260`
- `action.yml:310`

### github-env-injection (severity: high)

Multiple `run:` blocks write attacker-controlled values derived from `inputs.*` expressions to GitHub special environment files (`$GITHUB_ENV`, `$GITHUB_PATH`, `$GITHUB_OUTPUT`) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). This allows an attacker to inject arbitrary environment variables or output values by embedding newlines in the input.

1. **"Set atmos cli config path vars"** step: `${{ inputs.atmos-config-path }}` is written directly to `$GITHUB_ENV` via `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV`. No sanitization is applied.
2. **"Define Job Variables"** step: Values derived from `${{ inputs.stack }}`, `${{ inputs.component }}`, and `${{ inputs.sha }}` are written to `$GITHUB_OUTPUT` (e.g. `echo "stack_name=${STACK_NAME}" >> $GITHUB_OUTPUT`) without sanitization. The intermediate shell variables (`STACK_NAME`, `COMPONENT_NAME`) do not sanitize newlines.

Locations:

- `action.yml:97`
- `action.yml:260`

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

Fixed all security findings in action.yml:

1. **unpinned-uses**: Pinned all 9 unpinned action references to full 40-character commit SHAs with tag comments: actions/checkout@v4, cloudposse/github-action-setup-atmos@v2, cloudposse/github-action-atmos-get-setting@v2, aws-actions/configure-aws-credentials@v4 (×2), actions/cache@v4, cloudposse/github-action-terraform-plan-storage@v1 (×2), infracost/actions/setup@v3, actions/upload-artifact@v4.

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }} and ${{ github.repository_owner }}, ${{ github.event.repository.name }} expressions from run: blocks into env: blocks across all affected steps: 'Set atmos cli config path vars', 'Add Terraform and OpenTofu to Aqua', 'Define Job Variables', 'Atmos Terraform Plan', 'Generate Infracost Diff', 'Store Component Metadata to Artifacts', and 'Publish Summary or Generate GitHub Issue Description for Drift Detection'.

3. **github-env-injection**: Added sanitization (printf '%s' | tr -d '\n\r') before writing values to $GITHUB_ENV in 'Set atmos cli config path vars' and before writing all values to $GITHUB_OUTPUT in 'Define Job Variables'.

