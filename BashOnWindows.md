# Bash on Windows

There are a couple of options for running bash on Windows:
- Git Bash
- Ubuntu Bash

## Install Microsoft Terminal

This can be installed either through the Microsoft Store or via chocolatey
`> choco install microsoft-windows-terminal`


## Git Bash

Git Bash emulates a bash terminal on Windows to provides access to unix commands and the Git CLI.
It comes included as part of the [Git for Windows](https://gitforwindows.org/) install.

Git Bash can be added to Windows Terminal by going to Settings > Add a new profile  
Then click \+ New empty profile
- Name: Git Bash
- Command line: %ProgramFiles%\Git\git-bash.exe
- Icon: %ProgramFiles%\Git\mingw64\share\git\git-for-windows.ico
- Tab Title: Git Bash


## Linux Bash

Turn on Windows Subsystem for Linux

Go to Control Panel > Programs > Turn Windows features on or off  
And ensure Windows Subsystem for Linux is turned on.

Install the latest Ubuntu LTS (long term support) version from the Microsoft Store (or your preferred distro, but these instructions will use Ubuntu).

There are different ways of opening the Ubuntu shell terminal:
- From Microsoft Store
- Ubuntu shortcut in the Start menu
- `ubuntu2004` (the 20.04 distro version) from powershell or cmd
- `wsl` or `bash` from powershell or cmd
- From Windows Terminal - access is automatically added once Ubuntu is installed


### Bash Commands

`wslfetch` provides details about WSL, this is built into the Ubuntu WSL distro.
Other distros need the WSLU package to be installed.


### Keeping Ubuntu up-to-date

Ubuntu can be kept up-to-date in the terminal by doing the following:
- `sudo apt update` to check for package updates
- `apt list --upgradable` to list all packages that can be upgraded 
- `sudo apt upgrade` to upgrade all the packages