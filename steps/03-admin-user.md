# Step 03 — Admin user

## Why
Replace the generic `debian` user with a named admin account.
- Clearer audit trail (who did what)
- Allows later locking down `debian` or root entirely
- Standard Linux practice: one human, one named account

## What I did
```bash
sudo adduser haythem
sudo usermod -aG sudo haythem
groups haythem            # confirmed: haythem sudo users
su - haythem
sudo whoami               # returned: root → sudo works
```

## Result
- User `haythem` exists with home `/home/haythem`
- Member of `sudo` group → can `sudo` with own password
- Tested and confirmed