```markdown
# Suspend/Resume Regression in systemd v258

## Overview
This repository documents a reproducible regression introduced in **systemd v258**.  
Affected systems enter suspend normally, but upon resume, `systemd-logind` immediately re‑triggers suspend within ~20–30 seconds.  
This results in a **double suspend loop**: suspend → resume → suspend again.

The regression has been confirmed across multiple distributions (Fedora, Arch, CachyOS, Ubuntu, Debian, openSUSE).  
Upstream bug report: [systemd issue #40078](https://github.com/systemd/systemd/issues/40078)

---

## Symptoms
- System suspends after idle timeout as expected.  
- On resume, the system suspends again almost immediately without user activity.  
- Loop continues until interrupted by user input or configuration changes.

---

## Bisect Results
- **Last good commit:** `6833cdfa04` (tags/v258~1)  
- **First bad commit:** `781d9d0789` (v258 release commit)  
- Regression introduced exactly at the v258 release commit.

---

## Build & Test Method
```bash
cd ~/systemd
git checkout <commit>
sudo rm -rf build
CFLAGS="-Wno-error=flex-array-member-not-at-end" meson setup build
ninja -C build
sudo ninja -C build install
```

- Kernel tested: `6.18.0-3-cachyos`  
- Architecture: `x86_64`  
- Distribution: CachyOS (Arch‑based rolling release)

---

## Logs

**Bad commit (`781d9d0789`):**
```
Dec 13 19:21:27 systemd-logind[774]: Operation 'suspend' finished.
Dec 13 19:21:52 systemd-logind[774]: The system will suspend now!
```

**Good commit (`6833cdfa04`):**
```
Dec 13 19:31:03 kernel: Low-power S0 idle used by default for system suspend
Dec 13 19:31:03 kernel: nvme 0000:04:00.0: platform quirk: setting simple suspend
```

---

## Cross‑Distro Reports
This regression is not isolated. Similar issues have been reported across multiple distributions:

- **Fedora**  
  - [Discussion: Occasional failure to enter Suspend](https://discussion.fedoraproject.org/t/ocassional-failure-to-enter-suspend-after-which-becomes-pc-non-responsive/141390)

- **Arch Linux / derivatives**  
  - [Arch Linux Forums: suspend/resume issues](https://bbs.archlinux.org/viewtopic.php?id=300801)

- **Ubuntu**  
  - [AskUbuntu: suspend only works once, second time fails](https://askubuntu.com/questions/1540652/update-ubuntu-suspends-only-once-second-time-it-fails-and-then-fails-to-shut)

- **Debian**  
  - [Debian Forums: system suspends despite settings](https://forums.debian.net/viewtopic.php?p=794725)

- **openSUSE**  
  - [Unix.SE: System goes to suspend mode when it is not supposed to](https://unix.stackexchange.com/questions/788917/system-goes-to-suspend-mode-when-it-is-not-supposed-to)

---

## Workarounds
- **Downgrade to systemd 257.x**: avoids the regression.  
- **Disable IdleAction** in `logind.conf`: prevents automatic suspend, but removes idle suspend functionality.  
- **Manual suspend only**: use `loginctl suspend` until upstream fix is released.

---

## Progress / Status
- ✅ Regression identified and bisected  
- ✅ Upstream bug filed: [systemd issue #40078](https://github.com/systemd/systemd/issues/40078)  
- 🔄 Awaiting upstream maintainer triage and fix  
- 🔄 Monitoring related distro reports  
- ⏳ Patch expected in v259 or later; will test once available  

---

## FAQ
**Is this a distro bug?**  
No. It’s an upstream systemd regression introduced in v258. All distros shipping v258 are affected.  

**Does downgrading help?**  
Yes. Downgrading to systemd 257.x avoids the bug.  

**When will it be fixed?**  
Upstream maintainers are aware (issue #40078). Fix expected in v259 or later.  

**What can I do to help?**  
Add your distro/version details and logs to the upstream issue. This helps confirm scope and urgency.

---

## Purpose of this Repo
This repository serves as:
- A **reference point** for users hitting the same bug.  
- A **technical log** of bisect results, build steps, and logs.  
- A **status tracker** until upstream resolves the regression.

- 

---

```
```
# Show only suspend/resume events
journalctl -b | grep -i suspend

# Show only logind activity
journalctl -b | grep -i logind

# Show kernel power management events
journalctl -b | grep -i PM:

# Single command to show single command that will give you just the suspend/resume loop evidence, including both the kernel events and the logind triggers
journalctl -b | grep -E "The system will suspend now|Operation 'suspend' finished|PM: suspend"

```
After resuming from suspend, systemd-logind repeatedly triggers new suspend cycles every ~20s:

Dec 15 04:53:56 kev kernel: PM: suspend exit
Dec 15 04:53:56 kev systemd-logind[790]: Operation 'suspend' finished.
Dec 15 04:54:17 kev systemd-logind[790]: The system will suspend now!
Dec 15 04:54:19 kev kernel: PM: suspend entry (s2idle)
Dec 15 04:54:24 kev kernel: PM: suspend exit
Dec 15 04:54:24 kev systemd-logind[790]: Operation 'suspend' finished.
Dec 15 04:54:47 kev systemd-logind[790]: The system will suspend now!
Dec 15 04:54:49 kev kernel: PM: suspend entry (s2idle)
Dec 15 04:54:54 kev kernel: PM: suspend exit
Dec 15 04:54:54 kev systemd-logind[790]: Operation 'suspend' finished.
Dec 15 04:55:16 kev systemd-logind[790]: The system will suspend now!
Dec 15 04:55:18 kev kernel: PM: suspend entry (s2idle)
Dec 15 04:55:24 kev kernel: PM: suspend exit
Dec 15 04:55:24 kev systemd-logind[790]: Operation 'suspend' finished.
Dec 15 04:55:47 kev systemd-logind[790]: The system will suspend now!
Dec 15 04:55:49 kev kernel: PM: suspend entry (s2idle)
Dec 15 04:55:53 kev kernel: PM: suspend exit
Dec 15 04:55:53 kev systemd-logind[790]: Operation 'suspend' finished.
Dec 15 04:56:15 kev systemd-logind[790]: The system will suspend now!
Dec 15 04:56:17 kev kernel: PM: suspend entry (s2idle)
Dec 15 04:56:22 kev kernel: PM: suspend exit
Dec 15 04:56:22 kev systemd-logind[790]: Operation 'suspend' finished.
Dec 15 04:56:43 kev systemd-logind[790]: The system will suspend now!
Dec 15 04:56:46 kev kernel: PM: suspend entry (s2idle)
Dec 15 04:56:53 kev kernel: PM: suspend exit
Dec 15 04:56:53 kev systemd-logind[790]: Operation 'suspend' finished.

...

This does not occur on systemd v257. Regression introduced in v258.

