# FILE CORRUPTER LAB 001 — Linux Privilege Escalation

**THINK. HACK. DEFEND.**

Intermediate | 45–60 minutes | Linux | v1.0

## 1. Mission
Understand how Linux enforces privilege boundaries, recognize unsafe permissions, and verify remediation. Privilege escalation occurs when a process or user gains permissions beyond those intended. Vertical escalation crosses privilege levels; horizontal escalation crosses peer account boundaries.

## 2. Laboratory requirements
Use a disposable Ubuntu or Debian virtual machine you own, with a regular user account and an authorized administrator account. Take a snapshot. No external targets are needed. Run the exercises as a regular user unless explicitly indicated. Never enter real credentials into evidence notes.

## 3. Establish your identity (5 minutes)

```bash
whoami
id
groups
```

Record your UID, primary GID and supplementary groups. UID 0 is root. Group membership can grant access independently of file ownership. A username alone does not establish effective privileges.

## 4. Understand permissions (15 minutes)
Create a harmless local training fixture:

```bash
mkdir -p "$HOME/fc-lab-001"
cd "$HOME/fc-lab-001"
printf 'Synthetic training record\n' > record.txt
chmod 644 record.txt
ls -l record.txt
stat -c '%A %a %U %G %n' record.txt
```

Expected mode: `-rw-r--r--` (644). Owner can read/write; group and others can read. Read=4, write=2, execute=1. For directories, execute permits traversal and write generally permits entry creation/deletion subject to other controls.

Simulate an overly permissive file, then remediate it:

```bash
chmod 666 record.txt
stat -c '%A %a %n' record.txt
chmod 600 record.txt
stat -c '%A %a %n' record.txt
```

Expected modes: 666, then 600. The first grants write access to all permission classes; the second restricts reading and writing to the owner. These commands affect only the synthetic fixture. A world-writable file is not automatically a root escalation: its impact depends on which privileged processes trust it.

## 5. Audit privileged execution (10 minutes)

```bash
sudo -l
find /usr/bin -maxdepth 1 -type f -perm -4000 -print
find /usr/bin -maxdepth 1 -type f -perm -2000 -print
```

`sudo -l` lists permitted sudo commands for the current user; authentication may be required. If denied, record that result. Do not execute listed privileged commands for this exercise. SUID executables can run with the file owner's effective UID; SGID executables can run with the file group's effective GID, subject to OS restrictions. Their presence is not itself evidence of a vulnerability. Compare results against a known-good system baseline. Do not remove special bits from system binaries indiscriminately.

## 6. Inspect a simulated trust boundary (10 minutes)

```bash
printf '#!/bin/sh\nprintf "Training only\\n"\n' > maintenance.sh
chmod 777 maintenance.sh
ls -ld . maintenance.sh
chmod 700 maintenance.sh
chmod 700 .
stat -c '%A %a %n' . maintenance.sh
```

The script is deliberately harmless and is never executed with elevated privileges. Imagine a privileged maintenance service consuming a file writable by ordinary users. That creates a trust-boundary risk. Restricting file and directory permissions reduces the risk; in a real service, administrators must also verify trusted ownership, parent directories, configuration and execution context. The exercise does not create or exploit a privileged service.

## 7. Defensive investigation (5 minutes)
On a system using systemd, inspect available sudo logs:

```bash
journalctl -t sudo --since today --no-pager
```

Access and output vary by distribution and logging configuration. Empty output is not proof that no sudo activity occurred. Administrators can review authentication logs, changes to privileged files, unexpected group memberships and service configuration changes. Preserve evidence before modifying a suspected compromised system.

## 8. Knowledge assessment
1. What does UID 0 represent?
2. Why is mode 666 risky for a sensitive file?
3. What is the difference between SUID and SGID?
4. Does a SUID binary automatically indicate a vulnerability?
5. Why must a privileged service's parent directories also be protected?

### Answer key
1. The root user identity.
2. All permission classes have read/write access, subject to additional controls.
3. SUID changes effective user identity; SGID changes effective group identity on eligible executables.
4. No; intended behavior, ownership, integrity and configuration matter.
5. Writable directories can permit replacement or redirection of trusted files.

## 9. Completion checklist
- [ ] Recorded identity and group membership.
- [ ] Compared modes 644, 666 and 600.
- [ ] Audited sudo and special-bit executables without exploiting them.
- [ ] Remediated the simulated maintenance file and directory.
- [ ] Answered all five assessment questions.

## 10. Cleanup
Remove only the two synthetic files and then the empty training directory:

```bash
cd "$HOME"
rm -f -- "$HOME/fc-lab-001/record.txt" "$HOME/fc-lab-001/maintenance.sh"
rmdir -- "$HOME/fc-lab-001"
```

## Further reading
- `man 1 chmod`, `man 1 id`, `man 1 find`, `man 8 sudo`, `man 7 credentials`
- [Linux man-pages: credentials](https://man7.org/linux/man-pages/man7/credentials.7.html)

## Responsible use
Use only systems you own or have explicit authorization to assess. This lab teaches auditing and remediation using synthetic fixtures, not obtaining unauthorized root access.

Copyright © 2026 FILE CORRUPTER LLC.

[Website](https://filecorrupter.org) | [X](https://x.com/filecorrupter)

**THINK. HACK. DEFEND.**
