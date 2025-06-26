---
categories: [Pre Security]
---

# Linux Fundamentals - Part 1

## What is Linux?

Linux is an operating system like Windows and macOS.  
It is widely used in servers, smart cars, Android devices, and more.  
Linux is lightweight, faster than Windows, and open-source.

Most Linux systems don’t have a GUI (Graphical User Interface) and instead rely on the command line in the terminal.  
It might be intimidating at first, but once you're used to it, it's very efficient and powerful.

---

## Fundamental Commands

| Command | Description |
|---------|-------------|
| `echo` | Prints the text you enter. <br> Example: `echo hi` → `hi` <br> For multiple words, use double quotes: `echo "G'day mate"` → `G'day mate` |
| `whoami` | Shows the current logged-in user. |
| `ls` (list) | Lists files and folders in the current directory. |
| `cd` (change directory) | Changes the current working directory. |
| `cat` (concatenate) | Displays the contents of a file. |
| `pwd` (print working directory) | Prints the full path of the current directory. |

---

## File Searching

### find

Use `find` to locate files by name or extension.

- Find a file by name:
  find -name note.txt
- Find all .txt files:
  find -name *.txt
![imagie](/assets/find.png)


### grep
Search for specific content inside a file.

Example: Search for a specific IP in access.log
grep "81.143.211.90" access.log

### Shell Operators
&	Runs a command in the background.
&&	Runs the second command only if the first one is successful.
Example: command1 && command2
>	Redirects output to a file (overwrites if the file exists, creates if it doesn't).
>>	Appends output to the end of a file without overwriting.
![imagie](/assets/command.png)