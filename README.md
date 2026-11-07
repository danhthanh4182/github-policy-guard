# GitHub Policy Guard (`github-policy-guard`)

> **Automated anti-malware, anti-PUA, and security compliance skill for AI Coding Agents.**  
> Prevent coding agents from generating code patterns, scripts, or release artifacts that violate GitHub Acceptable Use Policies or trigger automated false-positive suspensions.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Skills.sh](https://img.shields.io/badge/skills.sh-indexed-success.svg)](https://skills.sh)

---

## 🎯 Purpose

AI coding agents (Codex, Claude Code, Cursor, OpenCode, Copilot, etc.) frequently write automation scripts and system utilities. However, certain code patterns can inadvertently trigger **GitHub's automated abuse/malware detection systems**, leading to account flags and repository suspensions:
- Running command shells in hidden mode (`WScript.Shell.Run ..., 0, False`) flagged as dropper behavior (MITRE ATT&CK T1059.005).
- Privilege elevation using `Invoke-Expression` with dynamic strings.
- Staging self-signed certificates, private keys (`.pfx`, `.key`), or password files.
- Committing precompiled binary executables (`.exe`, `.dll`) with zero reputation.
- Hardcoding user paths (`C:\Users\<Name>\...`).

`github-policy-guard` installs strict guardrails into your agent's context, preventing these patterns from ever being written.

---

## 🚀 Quick Install via `skills.sh`

Install this skill into your local AI agent environment using the `skills.sh` CLI:

```bash
# Add to your current project or global agent environment
npx skills add <your-username>/github-policy-guard
```

Supported agent runtimes:
- **Codex CLI**: `~/.codex/skills/`
- **Claude Code**: `~/.claude/skills/` or `~/.agents/skills/`
- **OpenCode**: `~/.opencode/skills/`
- **Pi / Oh My Pi**: `~/.pi/skills/` and `~/.omp/agent/skills/`

---

## 🛡️ Core Rules Enforced

1. **Zero Stealth Execution**: Prohibits hidden windows (`0 = SW_HIDE`, `WScript.Shell.Run ..., 0, False`, or unprompted `-WindowStyle Hidden`).
2. **Zero Dynamic Privilege Execution**: Prohibits `Invoke-Expression` or `eval()` with dynamic strings. Mandates direct parameterized execution (`& executable @args`).
3. **Zero Secrets in Git**: Enforces `.gitignore` rules for `.pfx`, `.key`, `.pem`, and `*pass*.txt` before keys are generated.
4. **Zero Tracked Binaries**: Prohibits committing `.exe`, `.dll`, or `.zip` files to Git. Directs all releases through GitHub Actions CI/CD with verifiable Artifact Attestations.
5. **No Hardcoded User Paths**: Enforces dynamic path resolution (`$env:USERPROFILE`, `dirs::data_dir()`).
6. **No Deceptive PUA Patterns**: Forbids uncoordinated disabling of OS security features. Requires precise technical terminology.
7. **Safe Rollback**: Requires `--dry-run` inspection and full `restore` capabilities for any system/registry modifications.

---

## 📄 License
Released under the [MIT License](LICENSE).
