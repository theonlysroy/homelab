## Content
- [Handling laptop lid](#handling-laptop-lid)
- [User setup](#user-setup)

---
---

### Handling laptop lid

To access the server while the laptop lid is closed, we need to configure what lid closing behaviour. 

Update `/etc/systemd/logind.conf`. Remove `#` from the line start and set the value as `ignore`
```bash
HandleLidSwitch=ignore
```

### User setup

To add a new user and optionally allow them `sudo` permissions. Login with `root` user to perform these actions.
```bash
# update NEW_USER with actual user name
NEW_USER=admin
adduser $NEW_USER
usermod -aG sudo $NEW_USER
```
