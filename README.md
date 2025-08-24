# Digital Forensics Case Study — Lenovo T490 (CAINE)

**Date prepared:** 2025-08-24  
**Author:** 0wnleahshad0w (alias)

## Overview
Objective: identify residual artifacts from any **prior users** of a Lenovo ThinkPad T490 acquired in early August 2025. Work performed in the CAINE forensic Linux environment (Starkane).

Focus areas:
- Mounting and verifying the Windows volume
- Prefetch / Registry review (execution & install timelines)
- Attempted **SAM/SYSTEM** hive extraction (NTLM hashes)
- Attempted deployment of **Autopsy** (Sleuthkit case manager)

**Note:**  
Some “Expected screenshots” listed below are workflow template placeholders.  
Only a subset were captured during this case; others are left in place to show the intended documentation style.


## Environment
- **Hardware:** Lenovo ThinkPad T490
- **Live OS:** CAINE 14 (Starkane)
- **Evidence drive:** SanDisk Extreme 2TB (EXT4, read‑only use for E01 set)
- **Working storage:** LaCie 5TB (APFS on macOS)
- **Windows mount point:** `/mnt/t490`

## Methods & Steps

### 1) Mount & Verify Evidence
- Identified Windows partition and mounted read‑only to `/mnt/t490`
- Confirmed registry hives present in `Windows/System32/config` (`SAM`, `SYSTEM`, `SOFTWARE`, `SECURITY`)

_Expected screenshots:_  
- `screenshots/mount_success.png`  
- `screenshots/registry_hives_present.png`

### 2) Prefetch (Execution Timeline)
- Listed `C:\Windows\Prefetch` and reviewed timestamps/filenames
- Earliest activity around **2025‑07‑31** (likely refurb/store prep)
- Majority of activity **2025‑08‑10+** (post‑acquisition)

_Expected screenshots:_  
- `screenshots/prefetch_listing.png`  
- `screenshots/timeline_prefetch_start.png`

**Interpretation:** No visible 2024 execution artifacts → consistent with **reset/reimage prior to resale**.

### 3) Registry Analysis (RegRipper)
Commands:
```bash
rip.pl -r /mnt/t490/Windows/System32/config/SOFTWARE -p uninstall > logs/registry_uninstall.txt
rip.pl -r /mnt/t490/Windows/System32/config/SYSTEM   -p networklist > logs/regripper_networklist.txt
```
Findings:
- Install artifacts dated **Aug 2025**: NVIDIA driver, Microsoft 365, Python, VirtualBox, MSVC runtimes
- Network profiles match a new deployment window
- No legacy profiles beyond built‑ins + current alias account

_Expected screenshots:_  
- `screenshots/regripper_uninstall.png`  
- `screenshots/regripper_networklist.png`

### 4) SAM/SYSTEM Hash Extraction Attempts
**a) samdump2**  
- Attempted install from old .deb and source tarball; compile fails against OpenSSL 3.x (removed `DES_*` APIs) → **not viable** on CAINE 14 without patching/old OpenSSL.

**b) Impacket `secretsdump`**
```bash
python3 -m impacket.examples.secretsdump   -sam /mnt/t490/Windows/System32/config/SAM   -system /mnt/t490/Windows/System32/config/SYSTEM   LOCAL | tee /home/caine/sam_hashes.txt
```
- File created but **0 bytes** (no NTLM lines) → likely clean reimage/reset or inaccessible hive state under live mount

_Expected screenshots:_  
- `screenshots/impacket_attempt.png`  
- `screenshots/hash_dump_empty.png`

### 5) Autopsy (Sleuthkit) Attempts
- Downloaded Autopsy 4.22.x; installed Sleuthkit libs, Java 17; repeated `apt --fix-broken`/manual `.deb` installs
- Crashed at “Turning on modules…”; JNI / `libtsk_jni.so` issues and JavaFX module warnings

_Expected screenshots:_  
- `screenshots/autopsy_error_modules.png`

**Conclusion for Autopsy:** On CAINE 14, dependency drift makes Autopsy unstable. Prefer a known‑good Ubuntu/Windows VM for Autopsy cases.

## Key Findings
- **No 2024 user artifacts** in Prefetch or Registry
- System state consistent with a **factory reset or refurb image** before sale
- Hash extraction produced **no usable output**
- Autopsy not stabilized within reasonable effort on live distro

## Limitations
- Live‑distro package drift (Java/Sleuthkit versions)
- Legacy tools (samdump2) incompatible with modern OpenSSL
- Potential hive state/locking under live mount

## Lessons Learned
- DFIR requires **process discipline**: document attempts and obstacles
- Prefetch + Registry are often enough to show **reimage/timeline**
- For complex tooling (Autopsy), use a **stable VM** over a live distro

## Appendix: Representative Commands
```bash
# Verify hives present
ls -lh /mnt/t490/Windows/System32/config/{SAM,SYSTEM,SOFTWARE,SECURITY}

# Prefetch listing (first 40)
ls -lah --time-style=long-iso /mnt/t490/Windows/Prefetch | head -n 40

# RegRipper parsers
rip.pl -r /mnt/t490/Windows/System32/config/SOFTWARE -p uninstall > logs/registry_uninstall.txt
rip.pl -r /mnt/t490/Windows/System32/config/SYSTEM   -p networklist > logs/regripper_networklist.txt

# Secretsdump (show + save)
python3 -m impacket.examples.secretsdump   -sam /mnt/t490/Windows/System32/config/SAM   -system /mnt/t490/Windows/System32/config/SYSTEM   LOCAL | tee logs/secretsdump_stdout.txt
```

---

### Screenshot Safety Note
Only include screenshots that **do not** expose: real names/initials, emails, license keys, Wi‑Fi/MFA secrets, or security‑question answers. If in doubt, omit.

