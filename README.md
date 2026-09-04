# ✦ Codex × Termux Environment

> A compact, reproducible snapshot of the environment behind this repository.

<p align="center">
  <img alt="Platform" src="https://img.shields.io/badge/platform-Android-3DDC84?style=for-the-badge&logo=android&logoColor=white">
  <img alt="Architecture" src="https://img.shields.io/badge/arch-ARMv7-6E4DFF?style=for-the-badge&logo=arm&logoColor=white">
  <img alt="Shell" src="https://img.shields.io/badge/shell-Bash-121011?style=for-the-badge&logo=gnubash&logoColor=white">
</p>

<p align="center">
  <img alt="Node" src="https://img.shields.io/badge/Node.js-v26.4.0-339933?style=flat-square&logo=nodedotjs&logoColor=white">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.14.6-3776AB?style=flat-square&logo=python&logoColor=white">
  <img alt="GitHub CLI" src="https://img.shields.io/badge/GitHub_CLI-2.100.0-181717?style=flat-square&logo=github&logoColor=white">
</p>

## At a glance

| Layer | Snapshot |
| --- | --- |
| **Platform** | Android · Linux kernel `5.10.43-android12-9-g76bdd754c12c` |
| **Host / ABI** | ARMv7 / `armeabi-v7a` · `armv8l` machine reporting |
| **Terminal** | Termux `1.46.0+really1.45.0-1` · Bash |
| **Workspace** | `/data/data/com.termux/files/home` |
| **Git** | `2.55.0` |
| **GitHub CLI** | `2.100.0` |
| **Node.js / npm** | `v26.4.0` / `11.19.1` |
| **Python** | `3.14.6` |

## Execution envelope

```text
┌─────────────────────────────────────────────────────────┐
│  Codex agent                                             │
│  ├─ sandbox: workspace-write                             │
│  ├─ filesystem: home workspace + temporary directories   │
│  ├─ network: restricted unless approved                  │
│  └─ terminal: Termux on Android / ARM                    │
└─────────────────────────────────────────────────────────┘
```

### Sandbox boundaries

| Capability | Status |
| --- | :---: |
| Read workspace | ✅ |
| Write workspace | ✅ |
| Write temporary files | ✅ |
| Arbitrary host writes | ⛔ |
| Unrestricted network | ⛔ |

## Runtime toolbelt

```text
Android / Termux
      │
      ├── Bash
      ├── Git 2.55.0
      ├── GitHub CLI 2.100.0
      ├── Node.js 26.4.0  ── npm 11.19.1
      └── Python 3.14.6
```

## Notes

- This is an **environment inventory**, not a device profile.
- No tokens, authentication data, account identifiers, IP addresses, or device serials are recorded here.
- Version values are a point-in-time snapshot from **2026-09-04**.

---

<p align="center"><sub>Built in a sandboxed Codex session · ✨ private by design</sub></p>
