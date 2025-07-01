---
categories: [Pre Security]
---

# Linux Fundaental 2

## 1. What are Flags, Switches, and Arguments in Linux Commands?

- **Argument**: A value passed to a command that specifies input or a target.
  - Example: `cat file.txt` → `file.txt` is an argument.

- **Flag / Switch**: Options that modify the behavior of a command.
  - Start with `-` or `--`.
  - Example: `ls -l` → `-l` is a flag (also called a switch).

> In most cases, *flags* and *switches* are used interchangeably.

---

## 2. What Do `ls`, `--help`, and `man` Do?

- `ls`: Lists files and directories.
- `--help` or `-h`: Shows a short help message for the command.
- `man [command]`: Opens the manual (man) page with detailed documentation.

---

## 3. What Does `ls -l` Do and Why Focus on the First 3 Columns?

- `-l`: Stands for "long listing format". It shows detailed file info.
- The first three columns:
  1. **Permissions** (e.g. `-rw-r--r--`)
  2. **Link count**
  3. **Owner** (user who owns the file)

> These columns are key for understanding file security and ownership.

---

## 4. Are Flags and Switches Different?

No. **Flags** and **switches** refer to the same thing: options that modify a command’s behavior, such as `-a`, `-l`, or `--version`.

---

## 5. What Does `ls -lh` Mean?

- `-l`: Long listing format
- `-h`: Human-readable file sizes (e.g., `1.1K`, `2.5M`)
- Combined:
  ls -lh
Shows detailed file info with readable file sizes.
Short options like -l and -h can be combined as -lh.

## 6. What Is su and What Does -l Mean in This Context?

su: Switch user (default is root).
su -l: Switches to a login shell, fully loading the new user’s environment.

Equivalent to --login.

Note: -l in su means login shell, while -l in ls means long listing format. They are different even though they look the same.

## 7. What Are /etc, /var, /root, and /tmp?
Directory	Description
/etc	Configuration files (e.g., for users, services)
/var	Variable data: logs, mail, databases
/root	Home directory of the root user
/tmp	Temporary files; often cleared on reboot

