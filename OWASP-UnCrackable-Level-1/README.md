# OWASP UnCrackable iOS Level 1 – Write-up / Walkthrough

A reverse engineering walkthrough for the UnCrackable Level 1 iOS challenge from the OWASP Mobile Application Security (MAS) project. This report details the methodology used to bypass anti-debugging mechanisms and extract the hidden secret directly via Objective-C Runtime manipulation using Frida.

## Challenge Summary
* Objective: Find the hidden secret (flag) encrypted or obscured within the application.
* Platform: iOS (Jailbroken device / Environment).
* Tools Used: Frida, macOS Terminal, Hopper Disassembler (for initial structural mapping).
* Result: Success. The flag was dumped via console logs and forced onto the UI.
* Discovered Secret: i am groot!

---

## Step-by-Step Analysis & Exploitation

### 1. Static Reconnaissance
The analysis began by inspecting the compiled application binary (UnCrackable Level 1) using the command-line strings utility to check for plain-text hardcoded flags.

strings "UnCrackable Level 1.app/UnCrackable Level 1" | grep -iE "secret|flag|key" -C 3

Observation:
While the search revealed standard UI strings (Congratulations! You found the secret!!, Verification Failed.), the actual flag was nowhere to be found in the static binary. This confirmed that the secret is either decrypted dynamically at runtime or computed on the fly.

---

### 2. Identifying Anti-Debugging Protections
Attempting to spawn the application dynamically using Frida's default behavior (frida -f sg.vp.UnCrackable1...) resulted in an immediate crash:

Spawned sg.vp.UnCrackable1... Resuming main thread!
Process terminated

Root Cause: The application implements an aggressive, early-stage Anti-Debugging / Anti-Jailbreak mechanism (likely embedded within the native C main function or early constructors). It executes within milliseconds of process initialization—well before Frida's instrumentation engine can fully initialize the Objective-C runtime environment. 

However, launching the app manually by tapping its icon on the iPhone succeeded without triggering a crash. This indicated the defensive check specifically monitors for debugger attachment during the early spawn cycle.

---

### 3. Bypassing Defenses via Late Injection (Attach Mode)
To circumvent the early anti-debugging routine, a Late Injection (Attach) strategy was deployed:
1. The application was launched manually on the iOS device (allowing the early boot checks to pass safely).
2. Frida was then instructed to hook into the already running, active process using the -n flag:

frida -U -n "UnCrackable Level 1" -l exploit.js

This bypassed the startup defense bottleneck entirely.

---

### 4. Runtime UI Inspection & Heap Manipulation
Initial structural mapping of the binary indicated that the target ViewController held a reference to an element called theLabel. Instead of risking entanglement with potential native C obfuscated comparison functions (like strcmp or memcmp which run outside the Obj-C runtime), the exploitation turned toward UI Automation & Memory Inspection.

The finalized Frida script allocates a delay to ensure full view instantiation, switches context safely to the iOS mainQueue (required for UI thread interactions), scans the heap for the active ViewController instance, and directly reads its text property while forcing it visible on screen:

if (ObjC.available) {
    console.log("[*] Objective-C Runtime active. Initializing UI exploration...");
    try {
        setTimeout(function() {
            ObjC.schedule(ObjC.mainQueue, function() {
                ObjC.choose(ObjC.classes.ViewController, {
                    onMatch: function(vc) {
                        var label = vc.theLabel();
                        if (label) {
                            console.log("[+] Target label found on heap!");
                            label.setHidden_(0);
                            var uiColor = ObjC.classes.UIColor;
                            label.setTextColor_(uiColor.redColor());
                            label.setBackgroundColor_(uiColor.yellowColor());
                            console.log("[+] UI visual properties updated successfully.");
                            console.log("[🎯] SUCCESS! label.text = " + label.text());
                        }
                        var view = vc.view();
                        if (view) {
                            view.setNeedsLayout();
                            view.layoutIfNeeded();
                        }
                    },
                    onComplete: function() {}
                });
            });
        }, 1000);
    } catch (e) {
        console.log("[-] Error during memory manipulation: " + e);
    }
} else {
    console.log("[-] Objective-C environment unavailable.");
}

---

## Final Verification

Upon running the script via Attach mode, the Frida REPL immediately dumped the target properties from memory, revealing that the developer stored the raw flag inside the hidden label asset:

[iPhone::UnCrackable Level 1 ]-> 
[*] Objective-C Runtime active. Initializing UI exploration...
[+] Target label found on heap!
[+] UI visual properties updated successfully.
[🎯] SUCCESS! label.text = i am groot!

Flag Captured: i am groot!
