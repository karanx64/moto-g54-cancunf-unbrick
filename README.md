### *CAUTION : THIS IS NOT A GUIDE BUT A LOG OF MY JOURNEY TO REVIVE MY PHONE FROM A FIRM-BRICK STATE (MORE THAN SOFT-BRICK, LESS THAN HARD-BRICK) AND IS SUMMARISED USING CHATGPT.*


# Motorola XT2343-5 (cancunf) — Full Unbrick & Recovery Log

This document is a reconstruction of a real-world recovery process performed on a Motorola MT6855 (cancunf / XT2343-5) device.  
It includes fastboot, AVB, A/B slot repair, and blankflash/SP Flash Tool usage.


Follow it at your own risk. This is only for informational purpose.

(AGAIN, NOT A GUIDE)

## How I bricked it:
- unlocked bootloader and tried many custom ROMs.
- rooted using kerselsu and whatnot.
- reflashed stock ROM, relocked bootloader.
- worked well until I rebooted. 
- stuck on "no valid os"
- when connected to PC, it did not stay on or off, instead stuck on a bootloop.
- did not stay in fastboot or did not stay off.

## Fix in a nutshell:
- removed backcover and disconnected battery.
- it made it stay off when connected to PC.
- did a blankflash.
- now it was able to stay in fastboot.
- but bootloader was still locked and was unable to go to recovery or fastboot-d mode.


---

# 1. Initial Failure State

## Symptoms
- Device booted only into **FASTBOOT**
- No Android boot
- No recovery
- Fastboot message:
```

reason: No bootable A/B slot

```

## Slot state
```

slot-successful:_a: no
slot-successful:_b: no
slot-unbootable:_a: yes
slot-unbootable:_b: yes
slot-retry-count:_a: 7
slot-retry-count:_b: 7

```

## Bootloader state
```

securestate: flashing_locked
iswarrantyvoid: yes
current-slot: a

```

---

# 2. Key Observations Before Recovery

### efuseBackup warning
```

Blowing efuseBackup is not allowed, skipping

```
✔ Safe — not related to brick

---

###  set_active failure
```

fastboot set_active a → FAILED
fastboot set_active b → FAILED

```

Indicates:
- A/B metadata corruption
- slot trust failure

---

### firmware mismatch clue
- Build fingerprint mismatch detected:
  - 5-5 vs 5-6 firmware variants

---

# 3. Partition & Flashing Attempts

## Verified partitions
```

boot_a / boot_b
vendor_boot_a / vendor_boot_b
vbmeta_a / vbmeta_b
dtbo_a / dtbo_b
lk, tee, scp, md1img, gz, vcp, gpueb

```

---

##  First super flash failure
```

Preflash validation failed
WARNING: vbmeta and vbmeta_system required before flashing super

```

---

# 4. Fix Applied — AVB Chain Repair

## Step 1: Flash AVB components
```

fastboot flash vbmeta_a vbmeta.img
fastboot flash vbmeta_b vbmeta.img
fastboot flash vbmeta_system_a vbmeta_system.img
fastboot flash vbmeta_system_b vbmeta_system.img

```

✔ Fixed validation chain

---

## Step 2: Flash super (sparse chunks)

```

fastboot flash super super.img_sparsechunk.0
fastboot flash super super.img_sparsechunk.1
...
fastboot flash super super.img_sparsechunk.21

```

✔ System partition restored

---

# 5. Boot Chain Restoration

## Flashed components

### Core boot chain
- boot_a / boot_b
- vendor_boot_a / vendor_boot_b
- dtbo_a / dtbo_b

### Firmware partitions
- lk_a / lk_b
- tee_a / tee_b
- scp_a / scp_b
- md1img_a / md1img_b
- spmfw / spmfw_b
- gz / vcp / gpueb / dpm

---

# 6. Critical Warning (efuseBackup)

```

Blowing efuseBackup is not allowed, skipping

```

✔ Safe to ignore  
✔ No hardware damage

---

# 7. GPT Flash Attempt

```

fastboot flash gpt PGPT

```

✔ Accepted by bootloader  
✔ No partition table issue

---

# 8. Slot State After Flashing

```

slot-unbootable:_a: no
slot-unbootable:_b: no
slot-successful:_a: no
slot-successful:_b: no
slot-retry-count:_a: 7
slot-retry-count:_b: 7

```

---

# 9. Recovery Breakthrough

After completing:

- vbmeta fix
- full super flash
- boot chain restore

Device transitioned:

✔ fastboot → fastbootd  
✔ fastbootd → Android boot  
✔ System boot restored

---

# 10. Final Working State

## Bootloader
```

securestate: flashing_locked
iswarrantyvoid: yes

```

## Android
- Axion AOSP booted successfully ✔
- A/B slots functional ✔
- Verified boot green ✔

---

# 11. Root Cause Analysis

### Primary issues:

1. Partial firmware mismatch
2. Broken AVB chain (vbmeta mismatch)
3. Incomplete super flashing
4. A/B slot corruption
5. Bootloader lock during inconsistent state

---

# 12. Critical Fix Summary

✔ Flash full boot chain (both slots)  
✔ Flash vbmeta BEFORE super  
✔ Flash vbmeta_system  
✔ Flash super via sparsechunks  
✔ Run fastboot -w if needed  
✔ Clear boot mode flags (fb_mode_clear)

---

# 13. Lessons Learned

## Motorola MTK A/B devices are strict:
- AVB must be consistent
- boot + vendor_boot must match build
- super must match vbmeta chain

---

## DO NOT:
- Lock bootloader on mismatched firmware
- Mix firmware versions across partitions
- Flash super without vbmeta chain

---

# 14. Blankflash Insight

Used SP Flash Tool / blankflash method earlier:

✔ likely restored preloader state  
✔ fixed low-level boot access  
✔ enabled fastboot recovery operations  

Not the main fix — but helped stabilize device access.

---

# Final Outcome

✔ Device fully recovered  
✔ Axion AOSP running  
✔ Bootloader unlocked  
✔ A/B slots healthy  
✔ Flashing stable  

---

# End of Log
