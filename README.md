# AD Group Member Adder

Command-line tool written in Python to add a user to an Active Directory group using LDAP and NTLM authentication.

Ideal for administrative tasks, permission automation, or integration into larger scripts.

## Features

- Secure connection to a domain controller via **NTLM**.
- Automatic lookup of group and user Distinguished Names (DN).
- Membership check to avoid duplicate entries.
- Atomic operation: adds user with an LDAP `MODIFY_ADD`.
- Clear success/error messages for troubleshooting.
- Simple, easy-to-modify codebase.

## Requirements

- Python **3.6** or higher.
- [ldap3](https://pypi.org/project/ldap3/) library installed.

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/ad-group-member-adder.git
   cd ad-group-member-adder
   ```

2. (Optional) Create and activate a virtual environment:
   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows: venv\Scripts\activate
   ```

3. Install the required dependency:
   ```bash
   pip install ldap3
   ```

## Usage

Run the script from the terminal with the required arguments:

```bash
python add_to_group.py -d domain.local -g "Group Name" -a username.to.add -u admin.user -p Password123
```

### Required Arguments

| Short | Long        | Description |
|-------|-------------|-------------|
| `-d`  | `--domain`  | Active Directory domain name (e.g., `company.com`). |
| `-g`  | `--group`   | Name of the group (**CN**) to which the user will be added. |
| `-a`  | `--adduser` | Username (**sAMAccountName**) of the user to add. |
| `-u`  | `--user`    | Username of an AD account with permission to modify group membership. |
| `-p`  | `--password`| Password for the administrative user. |

> **Security note:** Providing a password directly on the command line may expose it in shell history or process listings. For production environments consider using environment variables or secure configuration files.

### Example

Add user `john.doe` to the group `Developers` in the domain `contoso.local` using admin credentials:

```bash
python add_to_group.py -d contoso.local -g Developers -a john.doe -u admin -p "MyP@ssw0rd"
```

Expected output on success:

```
[+] Connected to Active Directory successfully.
[+] Group Developers found.
[+] User john.doe found.
[+] User added to group successfully.
```

If the user is already a member:

```
[+] Connected to Active Directory successfully.
[+] Group Developers found.
[+] User john.doe found.
[+] User is already a member of the group.
```

## How It Works

1. **LDAP Connection**: Uses `ldap3` to connect to the specified domain server with NTLM authentication.
2. **Group Search**: Queries LDAP for an object with `objectClass=group` and `cn=<group_name>`.
3. **User Search**: Queries LDAP for a user object matching the given `sAMAccountName`.
4. **Membership Check**: Compares the user's DN against the group's `member` attribute.
5. **Modification**: If not already a member, performs a `MODIFY_ADD` operation on the group's `member` attribute.

## Error Handling

The script prints descriptive messages and exits with code `1` in the following cases:

- Failure to bind/connect to the Active Directory server.
- Group not found.
- User not found.

If the modification fails (e.g., insufficient permissions, network issue) the message `[-] There was an error trying to add the user to the group.` is displayed but the script continues execution normally (return code could be improved for scripting).

