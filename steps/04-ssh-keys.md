# Step 04 — SSH key-based login

## Why
Replace password auth with public-key auth for the admin user.
- Passwords are brute-forceable; keys are not (with a passphrase).
- Foundation for disabling password auth and root login in the next step.

## What I did
On the VPS as `haythem`:
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
echo 'ssh-ed25519 AAAAC3...full key... haythem@HaythemSdn' >> ~/.ssh/authorized_keys
```

On my laptop:
```bash
cat ~/.ssh/id_ed25519.pub      # copied the public key
ssh haythem@<vps-ip>            # confirmed key login works
```

## Permissions reminder (SSH is strict)
- `~/.ssh` must be `700`
- `authorized_keys` must be `600`
- Both owned by the user, not root

## Always
- Keep the existing session open as a safety net when changing SSH config
- Use `>>` (append) not `>` (overwrite) when writing to `authorized_keys`