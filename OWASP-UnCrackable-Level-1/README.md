# OWASP UnCrackable iOS Level 1 – Write-up

A detailed reverse engineering walkthrough for the UnCrackable Level 1 iOS challenge from the OWASP Mobile Application Security (MAS) project. This report details the methodology used to bypass anti-debugging mechanisms.

---

## 📊 Challenge Summary

| Parameter | Value |
|-----------|-------|
| **Objective** | Find the hidden secret (flag) encrypted within the application |
| **Platform** | iOS (jailbroken device / test environment) |
| **Tools Used** | Frida, macOS Terminal, Hopper Disassembler |
| **Status** | ✅ Success |
| **Discovered Secret** | `i am groot!` |

---

## 🔍 Step-by-Step Analysis

### Step 1: Static Reconnaissance

**Goal:** Check the binary for hardcoded flags in plaintext

```bash
strings "UnCrackable Level 1.app/UnCrackable Level 1" | grep -iE "secret|flag|key" -C 3
```

**Observations:**
- ✓ Found UI strings: `Congratulations! You found the secret!!`, `Verification Failed.`
- ✗ Actual flag not present in the static binary
- ⚠️ Conclusion: Flag is dynamically generated/retrieved at runtime

**Implication:** Dynamic analysis required

---

### Step 2: Identifying Anti-Debugging Protection

**Problem:** Attempting to spawn the application with standard Frida

```bash
frida -f sg.vp.UnCrackable1...
```

**Result:**
```
Spawned sg.vp.UnCrackable1...
Resuming main thread!
Process terminated ❌
```

**Root Cause:**
- Application implements aggressive anti-debugging / anti-jailbreak mechanism
- Executes at early stage (native C main function or constructors)
- Detects debugger attachment during initialization

**Key Finding:**
✓ Manual app launch (tapping icon) **does NOT** crash
→ Protection is specifically triggered when spawned by debugger

---

### Step 3: Bypassing Defenses via Late Injection (Attach Mode)

**Strategy:** Circumvent startup protection by attaching instead of spawning

**Procedure:**
1. Manually launch the application on iOS device (allow startup checks to pass)
2. Attach Frida to the already-running process

```bash
frida -U -n "UnCrackable Level 1" -l exploit.js
```

**Results:**
- ✅ Startup checks pass without issues
- ✅ Frida attaches to running process
- ✅ Injected JavaScript executes successfully

---

### Step 4: Runtime UI Inspection & Heap Manipulation

**Approach:**
1. Binary structural mapping → identify `ViewController` and `theLabel`
2. Instead of risking entanglement with native code, scan the heap
3. Bypass obfuscated Objective-C code

**Final Frida Script Strategy:**
```javascript
// Pseudo-code
1. Delay          → wait for full view instantiation
2. Context Switch → safely switch to iOS mainQueue
3. Heap Scan      → find active ViewController
4. Memory Read    → extract text from label.text
```

---

## ✅ Final Verification

**Running the script in Attach Mode:**

```
Attaching...
[*] Objective-C Runtime active. Initializing UI exploration...
[iPhone::UnCrackable Level 1 ]-> [+] Target label found on heap!
[+] UI visual properties updated successfully.
[🎯] SUCCESS! label.text = i am groot!
```

### 🏁 Flag Captured

```
Discovered Secret: i am groot! ✓
```

---

## 🎯 Key Takeaways

| Challenge | Solution |
|-----------|----------|
| Anti-debugging at startup | Attach Mode instead of Spawn |
| Hidden secret in runtime | Heap scanning + UI inspection |
| Native code obfuscation | Bypass via runtime manipulation |

---

## 📚 Further Resources

- [Frida Documentation](https://frida.re/)
- [OWASP Mobile Security](https://owasp.org/www-project-mobile-top-10/)
- [Hopper Disassembler](https://www.hopperapp.com/)

---

*⚖️ Disclaimer: Developed strictly for educational and security research purposes.*
