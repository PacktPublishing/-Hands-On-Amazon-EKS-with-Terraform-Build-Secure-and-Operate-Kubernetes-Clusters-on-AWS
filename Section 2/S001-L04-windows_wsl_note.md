Hi everyone! I hope you are enjoying the course so far!

**This note is relevant only for those running on Windows**, and it applies to you no matter which toolchain path you pick later in the course. If you use the devcontainer, Docker Desktop runs its engine on WSL 2. If you install the tools yourself instead, WSL is where you will install and run them, since every command in the course is a Linux shell command. Either way, WSL comes first.

## Setting it up

It is one command. Open PowerShell **as administrator**, run it, and restart your machine when prompted:

```
wsl --install
```

That single command installs WSL from the Microsoft Store, enables the **Virtual Machine Platform** Windows feature that WSL 2 needs, and installs Ubuntu for you.

## If the Ubuntu install fails

Three things are worth checking, in this order:

1. **Hardware virtualization** has to be enabled in your BIOS or UEFI.
2. **Virtual Machine Platform** may need enabling by hand. Search the Start Menu for `Turn Windows features on or off`, tick the box for **Virtual Machine Platform**, and restart.
3. **Windows Subsystem for Linux**, in that same dialog, is a separate box from the one above. Current WSL 2 installs do not strictly require it, since it is the older component kept around for WSL 1. Even so, ticking it has fixed the install for a number of students in past runs of my courses, so if the first two steps did not get you there, tick it as well and restart. Leaving it enabled costs you nothing.

## If you already have WSL

Refresh it before you get going, since Docker Desktop needs WSL 2.1.5 or later:

```
wsl --update

wsl -l -v
```

The first command pulls the latest version, and the second confirms your distribution is running on version 2.

If you face any issues when setting up WSL in the upcoming lectures, don't hesitate to use the Q&A so that we can look for a solution together!
