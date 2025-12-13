# Suspend-Resume-Regression-Hunt-systemd-v258
Suspend-Resume-Regression-Hunt-systemd-v258


Here’s a compact recap you can save before rebooting — almost like a “primer” for when you come back:

---

## 📝 Primer: Suspend/Resume Regression Hunt (systemd v258)

### What we know
- **Bug**: After auto‑suspend, system re‑suspends ~15s after resume.  
- **Bisect**: Regression window is **between v257 (good)** and **v258 (bad)**.  
- **Release commit** (`781d9d0`) marked bad, but it only updates metadata → real bug is in preceding commits.

### What we’ve done
- Built and installed several suspect commits, but initially **did not reboot**, so those tests weren’t valid.  
- Clarified: **reboot is required after each install** because systemd is PID 1.  
- Established a **checklist** for testing each commit:
  1. `git checkout <commit>`
  2. Clean + build + install
  3. **Reboot**
  4. Run GNOME auto‑suspend test
  5. Mark commit **Good/Bad**

### Tested so far (without reboot → needs retest)
| Commit        | Description                                                   | Result (needs retest) |
|---------------|---------------------------------------------------------------|-----------------------|
| 4dddcce874    | machine: privilege checks (#38911)                            | Good (but no reboot)  |
| 310ab61139    | meson: link flags (#38901)                                    | Good (but no reboot)  |
| 778e95420a    | systemd-path: error handling                                  | Good (but no reboot)  |
| f82d80da06    | ansi-color: stack overflow fix                                | Good (but no reboot)  |

### Next suspects to test (with reboot)
- `119d332d9c` → machine privilege checks (variant)  
- `44e3c4c8bc` → machine: validate root directory over varlink  

---

## ✅ Action Plan After Reboot
1. Retest the commits already marked “good” — but this time reboot after install.  
2. Continue with `119d332d9c` and `44e3c4c8bc`.  
3. Keep updating your table with **Good/Bad** results.  
4. Once the first **Bad** commit is found, confirm by reverting it on top of v258.

---
