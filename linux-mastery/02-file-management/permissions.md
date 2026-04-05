# Linux Permissions

Each file has:

- Owner
- Group
- Others

## Permission Types

| Symbol | Meaning |
|-------|--------|
| r | Read |
| w | Write |
| x | Execute |

## Example
-rwxr-xr--

## Commands

'''bash
chmod 755 file.sh
chown user:group file.sh

## DevOps Use Cases
- Secure keys → chmod 400 key.pem
- Restrict configs → chmod 600 .en
