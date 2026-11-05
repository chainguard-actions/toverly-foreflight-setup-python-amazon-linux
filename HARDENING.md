<!-- markdownlint-disable -->

# Hardening Report: toverly-foreflight--setup-python-amazon-linux/df78ddc47690122c89f20f60c92891fffb801c68

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **toverly-foreflight--setup-python-amazon-linux/df78ddc47690122c89f20f60c92891fffb801c68** was hardened automatically. 12 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unsafe-shell (severity: high)

The 'Install uv' step downloads the uv installer and pipes it straight into bash: `curl -LsSf "https://github.com/astral-sh/uv/releases/download/${uv_version}/uv-installer.sh" | UV_UNMANAGED_INSTALL="${installation_directory}" bash --login`. The script should be saved to a file, checked, and then run as a separate command.

Locations:

- `action.yml:25`

### script-injection (severity: high)

Rule (a): the 'Find desired python version' step puts expressions straight into the shell: `desired_python_version=$(${GITHUB_ACTION_PATH}/find-desired-python-version.sh "${{ inputs.python-version }}" "${{ inputs.python-version-file }}")`. Rule (b): `${GITHUB_ACTION_PATH}` is also expanded without quotes.

Locations:

- `action.yml:32`

### script-injection (severity: high)

Rule (a): the 'Set installation directory' step puts an expression straight into the shell: `desired_python_version="${{ steps.find-desired-python-version.outputs.desired_python_version }}"`.

Locations:

- `action.yml:39`

### script-injection (severity: high)

Rule (a): the 'setup-python' step puts expressions straight into the shell: `installation_directory="${{ steps.set-installation-directory.outputs.installation_directory }}"` and `desired_python_version="${{ steps.find-desired-python-version.outputs.desired_python_version }}"`.

Locations:

- `action.yml:45`

### script-injection (severity: high)

Rule (a): the 'Add python to PATH' step puts an expression straight into the shell: `installation_directory="${{ steps.set-installation-directory.outputs.installation_directory }}"`.

Locations:

- `action.yml:53`

### script-injection (severity: high)

Rule (a): the 'Install pip' step puts an expression straight into the shell: `installation_directory="${{ steps.set-installation-directory.outputs.installation_directory }}"`.

Locations:

- `action.yml:62`

### github-env-injection (severity: high)

The 'Install uv' step writes a path built from the inherited `$HOME` variable to GITHUB_PATH with no sanitization: `echo "${installation_directory}" >> "${GITHUB_PATH}"`. The script never sets HOME itself, so the calling workflow controls it.

Locations:

- `action.yml:26`

### github-env-injection (severity: high)

The 'Find desired python version' step writes a value derived from inputs.python-version / inputs.python-version-file to GITHUB_OUTPUT with no `printf '%s' | tr -d '\n\r'` sanitization: `echo "desired_python_version=${desired_python_version}" | tee -a "${GITHUB_OUTPUT}"`.

Locations:

- `action.yml:33`

### github-env-injection (severity: high)

The 'Set installation directory' step writes a value derived from a step output and `$HOME` to GITHUB_OUTPUT with no sanitization: `echo "installation_directory=${HOME}/.setup-python-amazon-linux/.python-versions/${desired_python_version}" | tee -a "${GITHUB_OUTPUT}"`.

Locations:

- `action.yml:40`

### github-env-injection (severity: high)

The 'Add python to PATH' step writes a value derived from a step output to GITHUB_PATH with no sanitization: `echo "${installation_directory}/bin" >> "${GITHUB_PATH}"`.

Locations:

- `action.yml:54`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.python-version }}" appears directly in run: block of step "Find desired python version"; move to env: map

Locations:

- `action.yml:32`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.python-version-file }}" appears directly in run: block of step "Find desired python version"; move to env: map

Locations:

- `action.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unsafe-shell, script-injection, github-env-injection, static-inline-injection

**Notes:**

The uv installer is now downloaded to a mktemp file and checked (it must be non-empty and start with a '#!' line) before it runs with bash --login. UV_UNMANAGED_INSTALL is still passed the same way. All ${{ }} expressions (the two inputs and the step outputs) are now set in each step's env: block and used as quoted variables. The find-desired-python-version.sh path is now quoted. Its output is captured first and then stripped of newlines, so a failing script still stops the step. Every value written to GITHUB_PATH or GITHUB_OUTPUT now goes through printf '%s' | tr -d '\n\r' first.

