## Why key-only SSH auth over password

- Password auth is a brute-force attack surface regardless of `fail2ban`. Keys remove the attack vector entirely.
- Granular Access Control: Using ssh keys, specific device access can be easily revoked or authorized using the individual public keys on the server.
- Private keys can be encrypted with a `passphrase` thus making it more secure and less error-prone, as it requires both a `physical file` (stored on the disk) and a `passphrase` that only the user knows.


## Why a non-default SSH port

- Reduces log noise from automated scanners.
- Fewer login attempts make server spend less CPU and memory processing failed connection requests and running defensive tools like `fail2ban`.
