---
name: github-policy-guard
description: Enforce GitHub Acceptable Use Policies and anti-malware/anti-PUA hygiene across all AI coding agents. Use whenever writing scripts, Windows utilities, systems code (Rust/C++/Go/PowerShell/Bash), packaging releases, handling certificates or credentials, modifying OS/registry settings, or preparing Git repositories for GitHub.
---

# GitHub Policy Guard (Agent Safety & Compliance Skill)

This skill establishes mandatory guardrails to ensure that AI agents never write, refactor, or package code that violates GitHub's Terms of Service, Acceptable Use Policies (Malware, Exploits, PUA), or triggers automated security suspensions.

---

## 1. Zero-Tolerance Prohibitions (Never Generate These Patterns)

### 1.1. No Stealth Process Execution (MITRE ATT&CK T1059.005)
- **NEVER** use VBScript `WScript.Shell.Run ..., 0, False` or hidden window flags (`0 = SW_HIDE`) to conceal command prompts from users.
- **NEVER** invoke `powershell.exe -WindowStyle Hidden` or `cmd.exe /c start /b` without explicit user request or UI-facing tray requirements.
- **SAFE ALTERNATIVE:** Launch desktop applications using standard native GUI entrypoints, proper Windows subsystem PE flags (`windows_subsystem = "windows"` in Rust), or standard console wrappers.

### 1.2. No Arbitrary Dynamic String Execution (`Invoke-Expression`)
- **NEVER** pass concatenated or dynamic user/plan strings to `Invoke-Expression` or `iex`.
- **NEVER** use `eval()` or unquoted subshells in Bash/CMD to execute privileged system operations.
- **SAFE ALTERNATIVE:** Always use direct parameterized execution with parameter arrays:
  ```powershell
  # FORBIDDEN:
  # Invoke-Expression ("netsh int tcp set global " + $cmd)

  # REQUIRED:
  $args = @("int", "tcp", "set", "global") + ($cmd -split '\s+')
  & netsh.exe @args | Out-Null
  ```

### 1.3. No Hardcoded Secrets, Keys, or Plaintext Password Files
- **NEVER** write plaintext passwords to disk (e.g., `cert_pass.txt`, `password.txt`, `.env` committed to git).
- **NEVER** track private keys (`.pfx`, `.key`, `.pem`, `.pkcs12`) in the Git index.
- **MANDATORY RULE:** Before generating any cryptographic key, immediately add it to `.gitignore`:
  ```gitignore
  certs/
  *.pfx
  *.key
  *.pem
  *pass*.txt
  ```
- **SAFE ALTERNATIVE:** Read passwords exclusively from secure environment variables (`$env:CERT_PASSWORD`) or interactive secure string prompts.

### 1.4. No Pre-compiled Binary Commits (Avoid Zero-Reputation Executables)
- **NEVER** commit compiled binary executables (`.exe`, `.dll`, `.so`, `.zip`) directly into the Git repository.
- **NEVER** instruct users to upload locally compiled executables to GitHub Releases.
- **REQUIRED RULE:** All public release binaries must be built transparently via **GitHub Actions CI/CD** from public source code with verifiable GitHub Artifact Attestations (SLSA Provenance).

### 1.5. No Potentially Unwanted Application (PUA) / Tweaker Patterns
- **NEVER** disable OS security mechanisms, Windows Defender, SmartScreen, or Firewall without explicit defensive research authorization.
- **NEVER** use deceptive or sensationalist marketing claims (e.g., "Unlock 20% reserved bandwidth", "Bypass throttling").
- **REQUIRED RULE:** Use precise, documented technical terminology (e.g., *"Configures QoS NonBestEffortLimit Group Policy via documented registry keys"*).

### 1.6. No Hardcoded User Paths
- **NEVER** hardcode user profile paths like `C:\Users\Username\...`.
- **REQUIRED RULE:** Always use dynamic environment variables:
  - PowerShell: `Join-Path $env:USERPROFILE "AppData\Local\..."`
  - Rust: `dirs::data_local_dir()`, `dirs::home_dir()`, or `std::env::var("USERPROFILE")`
  - Python: `os.path.expanduser("~")` or `pathlib.Path.home()`

---

## 2. Mandatory Architectural Requirements

### 2.1. Idempotence, Dry-Run & Complete Rollback
Any tool or script that modifies the Windows Registry, network adapters, or system policies must:
1. Provide a `-n, --dry-run` flag so users and automated auditors can inspect every intended mutation without changing system state.
2. Automatically create a timestamped backup before mutation.
3. Provide a deterministic `restore` command that completely reverts all mutations back to standard Windows defaults.

### 2.2. Safe Installer & Uninstaller Design
In NSIS, InnoSetup, or MSI installers:
- **NEVER** execute unvalidated recursive directory deletion (e.g., `RMDir /r "$INSTDIR"` without verifying `$INSTDIR` is not root or empty).
- **ALWAYS** guard uninstaller logic:
  ```nsis
  ${If} $INSTDIR != ""
  ${AndIf} ${FileExists} "$INSTDIR\${APPEXE}"
    RMDir /r "$INSTDIR"
  ${EndIf}
  ```

### 2.3. Repository Compliance Documentation
Every public repository must contain:
1. `LICENSE`: Standard OSI-approved open-source license (e.g., MIT, Apache 2.0).
2. `SECURITY.md`: Clear vulnerability disclosure process.
3. `PRIVACY.md`: Explicit statement that the software contains zero spyware, zero telemetry, and zero credential harvesting.

---

## 3. Agent Verification Checklist Before Finishing Tasks
Before completing any coding turn that touches scripts, builds, or Git assets:
- [ ] No `.pfx`, `.key`, or `*pass*.txt` files staged in Git (`git status`).
- [ ] No `Invoke-Expression` in any PowerShell script.
- [ ] No hidden execution flags (`WScript.Shell.Run ..., 0, False`).
- [ ] No hardcoded user directories (`C:\Users\<Name>`).
- [ ] All binary releases are configured for GitHub Actions CI/CD.
