# Mobile Security & Reverse Engineering Portfolio

A collection of write-ups and functional scripts focusing on iOS runtime manipulation, security bypasses, and dynamic binary analysis.

## 🛠️ Tech Stack & Skills
* **Dynamic Analysis:** Runtime hooking and memory tampering via Frida.
* **Static Analysis:** Binary structure inspection using Hopper Disassembler.
* **iOS Internals:** Objective-C/Swift runtime exploration and heap manipulation.
* **Defensive AppSec:** Knowledge of RASP, anti-debugging, and Apple App Attest.

---

## 🚀 Solved Challenges

### 1. OWASP UnCrackable iOS Level 1
* **Challenge:** Extract a hidden secret from an iOS app.
* **Obstacle:** Active Anti-Debugging/Anti-Jailbreak mechanism causing crashes on boot.
* **Solution:** Bypassed startup checks using Frida's **Attach Mode (Late Injection)**. Scanned the heap for the active `ViewController` and extracted the secret string directly from the UI layer in memory.
* **Result:** Captured flag: `i am groot!`
* **Write-up & Code:** [Link to folder](./OWASP-UnCrackable-Level-1/)

---

## 📈 Methodology
1. **Recon:** Static binary mapping and symbol analysis.
2. **Analysis:** Tracking execution flow to pinpoint defensive hooks.
3. **Exploitation:** Developing lightweight Frida scripts for runtime memory patching.
4. **Remediation:** Documenting production-grade security fixes (OLLVM, App Attest).

---
*Disclaimer: Developed strictly for educational and security research purposes.*
