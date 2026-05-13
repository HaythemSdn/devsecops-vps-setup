# Step 02 — System update

## Why
Before any other changes, get to a fully patched baseline.

## What I did
```bash
sudo apt update
sudo apt list --upgradable    # was empty, image was current
sudo reboot                   # /var/run/reboot-required existed: new kernel on disk
```

## Result
- Running kernel: 6.12.86+deb13-amd64 (was 6.12.74 before reboot)
- No upgrades pending
- No reboot flag

## Lesson
After every `apt upgrade`, check `/var/run/reboot-required`. A new
kernel on disk doesn't protect anything until activated by reboot.