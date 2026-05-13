## Kernel patching mental model
- Kernel CVEs are mostly privilege-escalation bugs that turn a compromised
  www-data into root. For multi-tenant hosting, that = full takeover.
- Patching without rebooting = no protection.
- Reboots aren't dangerous when servers are configured properly; they're
  only dangerous when neglected.
- Build reboot frequency into the routine to keep the boot path tested.
- command to test the reboot : 
ls /var/run/reboot-required 2>/dev/null && echo "STILL NEEDED" || echo "No reboot needed"
