
# 🛠️ Fixing HP BIOS ACPI Errors on Linux (hp_bioscfg issue)  
  
## 💻 Device  
**HP 250 15.6 inch G10 Notebook PC**  
  
---  
  
## 📌 Problem  
  
On some HP laptops (including HP 250 G10), Linux may show ACPI errors in logs like:  

ACPI BIOS Error: AE_AML_BUFFER_LIMIT  
_SB.WMID.WQBZ / WQBE  
hp_bioscfg: Returned error 0x3

  
These errors are caused by the **HP BIOS configuration WMI driver (`hp_bioscfg`)**.  
  
---  
  
## 🎯 Root Cause  
  
- `hp_bioscfg` communicates with HP BIOS via WMI (Windows Management Instrumentation)  
- Some HP firmware versions (including G10 series) are buggy or incompatible with Linux  
- This causes ACPI method failures and kernel log errors  
  
---  
  
## ✅ Solution (Simple Fix)  
  
Remove the problematic kernel module.  
  
---  
  
## 1️⃣ Temporary Fix (Test)  
  
Unload the module:  
  
```bash  
sudo modprobe -r hp_bioscfg

----------

## 2️⃣ Verify Fix

Check system logs:

sudo dmesg -T | grep  -i acpi

If fixed, you should no longer see:

-   `AE_AML_BUFFER_LIMIT`
-   `hp_bioscfg` errors

----------

## 3️⃣ Permanent Fix (Recommended)

Blacklist the module so it does not load again:

echo  "blacklist hp_bioscfg" | sudo  tee /etc/modprobe.d/blacklist-hp-bioscfg.conf

----------

## 4️⃣ Rebuild initramfs

sudo update-initramfs -u

----------

## 5️⃣ Reboot

sudo reboot
```

# 🧠 If sensors stop working after fix

First understand:

## What We removed

We removed:

hp_bioscfg

This only affects:

-   HP BIOS configuration interface
-   some WMI sensor / hardware monitoring paths (on some models)


# 📊 What might break

After blacklisting it, you may lose:

-   ❌ `sensors` output (fan / temp missing)
-   ❌ HP BIOS settings tools (rare)
-   ❌ some advanced hardware telemetry

BUT:

-   ✔ CPU still works normally
-   ✔ system still stable
-   ✔ fan still controlled by BIOS (important)

----------

# ✅ Fix options (in order)

## 🔹 Option 1 (BEST): Use new driver you mentioned

Install HP WMI sensors driver:

sudo modprobe hp-wmi-sensors

Then:

sensors

If it works → DONE ✔

----------

## 🔹 Option 2: Re-enable hp_bioscfg (partial rollback)

If sensors completely break:

sudo  rm /etc/modprobe.d/blacklist-hp-bioscfg.conf  
sudo update-initramfs -u  
sudo reboot

----------

## 🔹 Option 3: Keep fix but restore partial functionality

Instead of full blacklist, we can:

-   keep `hp_wmi`
-   only load parts manually

(advanced tuning if needed)

----------

## 🔹 Option 4: Use standard Linux sensor stack (recommended fallback)

Install:

sudo apt install lm-sensors  
sudo sensors-detect

Then:

sensors

----------

# 🧠 Real-world truth (important)

On HP laptops like **HP 250 G10**:

👉 BIOS controls fan + thermal logic anyway  
👉 Linux sensors are often “read-only reporting layer”

So even if:

-   sensors disappear ❌  
    system is still safe

----------

# 👍 Recommended strategy

Do this order:

1.  Remove `hp_bioscfg` (fix ACPI errors)
2.  Try:
    

-   modprobe hp-wmi-sensors
    
-   If sensors OK → perfect
-   If not → install `lm-sensors`
-   Only rollback if you REALLY need HP tools
