# Mobile Security & Reverse Engineering Portfolio

A collection of write-ups and functional scripts focusing on iOS runtime manipulation, security bypasses, and dynamic binary analysis.

## 🛠️ Tech Stack & Skills

| Area | Description |
|------|-------------|
| **Dynamic Analysis** | Runtime hooking and memory tampering via Frida |
| **Static Analysis** | Binary structure inspection using Hopper Disassembler |
| **iOS Internals** | Objective-C/Swift runtime exploration and heap manipulation |
| **Defensive AppSec** | Knowledge of RASP, anti-debugging, and Apple App Attest |

---

## 🚀 Solved Challenges

### 1. OWASP UnCrackable iOS Level 1

**Objective:** Extract a hidden secret from an iOS application

**Challenge:** Active anti-debugging/anti-jailbreak mechanism causing crash on boot

**Solution:**
- Bypassed startup checks using **Attach Mode (Late Injection)** in Frida
- Scanned the heap for the active `ViewController`
- Extracted the secret string directly from the UI layer

**Result:** Flag: `i am groot!`

📖 **Documentation:** [Detailed write-up](./OWASP-UnCrackable-Level-1/README.md)

---

## 📈 Methodology

```
1. Recon         → Static binary mapping and symbol analysis
   ↓
2. Analysis      → Tracking execution flow to identify defensive hooks
   ↓
3. Exploitation  → Developing lightweight Frida scripts for memory patching
   ↓
4. Remediation   → Documenting production-grade security fixes
```

**Key Practices:**
- OLLVM - code obfuscation
- App Attest - application attestation
- Runtime obfuscation - runtime hardening

---

## 📋 Repository Structure

```
mobile-reverse-engineering-challenges/
├── README.md                          # This file
└── OWASP-UnCrackable-Level-1/
    ├── README.md                      # Detailed write-up
    ├── exploit.js                     # Frida script
    └── [additional resources]
```

---

*⚖️ Disclaimer: Developed strictly for educational and security research purposes.*
