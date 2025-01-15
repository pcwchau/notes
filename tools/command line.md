# Index

- [Index](#index)
- [Windows PowerShell command](#windows-powershell-command)
  - [General](#general)
      - [Run multiple commands](#run-multiple-commands)
      - [Find command location](#find-command-location)
      - [Environment variable](#environment-variable)
  - [Network](#network)
      - [Use `telnet` to connect to TCP socket](#use-telnet-to-connect-to-tcp-socket)
  - [SSH](#ssh)
      - [Use `ssh` to connect to a remote server](#use-ssh-to-connect-to-a-remote-server)
      - [Use `scp` to copy file to a remote server](#use-scp-to-copy-file-to-a-remote-server)
  - [Troubleshoot](#troubleshoot)
      - [.ps1 is not digitally signed](#ps1-is-not-digitally-signed)
- [Linux command](#linux-command)
  - [General](#general-1)
      - [Run multiple commands](#run-multiple-commands-1)
      - [Use `history` to search command](#use-history-to-search-command)
      - [Use `screen` to open multiple windows](#use-screen-to-open-multiple-windows)
      - [Change password](#change-password)
      - [Check linux version](#check-linux-version)
      - [Check environment variable](#check-environment-variable)
      - [Use `systemctl` to check service](#use-systemctl-to-check-service)
      - [Use `top` to check memory usage](#use-top-to-check-memory-usage)
  - [Read a file](#read-a-file)
      - [Use `less` to check log](#use-less-to-check-log)
      - [Use `tail` to check log (update real-time)](#use-tail-to-check-log-update-real-time)
  - [Write to a file](#write-to-a-file)
      - [Use `vi` to edit](#use-vi-to-edit)
  - [File operation](#file-operation)
      - [Use `ls`/`ll` to list directory content](#use-lsll-to-list-directory-content)
      - [Use `grep` to search in file](#use-grep-to-search-in-file)
      - [Use `find` to find file](#use-find-to-find-file)
      - [Use `rm` to remove file](#use-rm-to-remove-file)
      - [Use `tar` to archive file](#use-tar-to-archive-file)
  - [Manage package](#manage-package)
      - [Debian (eg. Ubuntu)](#debian-eg-ubuntu)

# Windows PowerShell command

## General

#### Run multiple commands

In `.bat`, run in sequence, one finish then run next
```bat
call ping google.com
call ping yahoo.com
```

In `.bat`, run in parallel, start at the same time
```bat
start "" ping google.com
start "" ping yahoo.com
start /b ping yahoo.com
```
- the `/b` option allows to execute the start command without creating a new window.

In `powershell`, use a semicolon to chain commands
```sh
ping google.com; ping yahoo.com
```

#### Find command location

In `powershell`, use `gcm` to find the location of, for example, the executable `npm`:

```sh
$ gcm npm

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Application     npm.cmd                                            0.0.0.0    C:\Program Files\nodejs\npm.cmd
```

#### Environment variable

In `powershell`:
```sh
# Show all variables
gci env:    # The short form of "Get-ChildItem -Path Env:\"

Name                           Value
----                           -----
ALLUSERSPROFILE                C:\ProgramData
JAVA_HOME                      C:\Program Files\Java\jdk-17.0.9

# Print a variable
PS C:\Users\peter.chau> $env:java_home

C:\Program Files\Java\jdk-17.0.9
```

## Network

#### Use `telnet` to connect to TCP socket

```sh
telnet localhost 15000
```

## SSH

#### Use `ssh` to connect to a remote server

```sh
ssh user@192.168.124.114
# Will prompt to enter password
```

#### Use `scp` to copy file to a remote server

```sh
# scp <source> <destination>
scp ./notificationservice-0.0.1-SNAPSHOT.jar user@192.168.124.114:/home/mfoperator/jars/copy_master_mt5
```

## Troubleshoot

#### .ps1 is not digitally signed

```sh
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

# Linux command

## General

```sh
CTRL + C  # send SIGINT to process, kill foreground process
          # can also discard the current command line and get a new prompt

CTRL + Z  # send SIGTSTP to process, suspend foreground process

CTRL + D  # send EOF to terminal, exit shell

CTRL + R  # reverse search, press CTRL + R for next result
```

#### Run multiple commands

```sh
# if left success, run right
cd /apps/logs && less console.log

# if left fail, run right
less console.log || less hktv-finp.log
```

#### Use `history` to search command

```sh
history

  976  cd /usr/src/jasperreport/buildomatic
  977  cd /usr/src/jasperserver/buildomatic/
  978  vi default_master.properties

!977    # for example, if we want to run the 977th command, type !977 and enter
```

#### Use `screen` to open multiple windows

```sh
# create a new screen
screen

# after entering a screen, you can create many windows
CTRL + A, C    # create a new windows
CTRL + A, N    # go to next window
CTRL + A, P    # go to previous window
CTRL + A, K    # kill the window
CTRL + A, D    # detach the screen
CTRL + A, ESC  # go to scrolling mode, press ESC again to leave


# check all active screens
screen -ls

There is a screen on:
        1156012.pts-29.tko-it-dev-gw    (Detached)
1 Socket in /run/screen/S-cwchau.

# go back to last screen
screen -r         # last screen
screen -r 1156012 # specific screen
```
- This command is useful when you want to do multiple tasks after entering a SSH environment.

#### Change password

```sh
passwd
```

#### Check linux version

```sh
cat /etc/lsb-release
cat /etc/*release
```

#### Check environment variable

```sh
printenv                # print all
printenv _JAVA_OPTIONS  # print specific
```

#### Use `systemctl` to check service

```sh
sudo systemctl start mysql
sudo systemctl stop mysql
sudo systemctl status mysql
```

#### Use `top` to check memory usage

```sh
top
```

## Read a file

#### Use `less` to check log

```sh
less /usr/local/tomcat/logs/catalina.out


UP          K   # go up one line
DOWN        J   # go down one line
PAGE UP     B   # go up one page (backward)
PAGE DOWN   F   # go down one page (forward)
G               # go to first line
SHIFT + G       # go to last line
Q               # quit

/ + <string>    # search forward from your current position and move you to the first found match
  - N           # find next match
  - SHIFT + N   # find previous match
```

#### Use `tail` to check log (update real-time)

```sh
tail -f -n1000 /usr/local/tomcat/logs/catalina.out
```
- `-f` : output appended data as the file grows
- `-n` : display the last N lines

## Write to a file

#### Use `vi` to edit

```sh
Enter input mode (leave input mode with Esc):
i  # insert before cursor
o  # open line below

In command mode:
dd        # delete line
u         # undo last change

Window motions:
CTRL + D  # scroll down (half a screen)
CTRL + U  # scroll up (half a screen)
CTRL + F  # page forward
CTRL + B  # page backward

CTRL + G         # show current line number
SHIFT + G        # go to last line
<n> + SHIFT + G  # go to line n

```

## File operation

#### Use `ls`/`ll` to list directory content

```sh
ls [options] [name ...]

ls -ltr               # list in time, earliest first, latest last
ls -al report*        # list file start with "RPT-MMS"
ls -al | grep report  # similar, list file containing "RPT-MMS"

ll                    # ls -l
```

- `-a` : list all files and directories (including hidden)
- `-l` : list in details
- `-t` : list in time order (descending)
- `-r` : list in reversed order

#### Use `grep` to search in file

```sh
grep [options] pattern [files]

grep hello file.txt
grep -r -n pattern dir/   # search in a directory, listing file name and line number
```

- `-n` : show line number (and file name if search in a directory)
- `-r` : search recursively in a directory

#### Use `find` to find file

```sh
find [directory-path] [filename] [options]

find / -name "TIB_js-jrs-cp_7.2.0_src.zip"  # find this file in whole system
find . -name "TIB_js-jrs-cp_7.2.0_src.zip"  # find this file in current directory
```

#### Use `rm` to remove file

```sh
rm -rf jasper*
```

- `-r` : remove recursively, including sub-directory
- `-f` : force remove, do not prompt any questions

#### Use `tar` to archive file

```sh
# Put all .jpg file into a new archive, then use gzip to zip it
tar -czf all.tar.gz *.jpg

# Use gzip to unzip the file, then extract the archive
tar -xzf all.tar.gz
```

- `-c` : Creates an archive file
- `-x` : Extracts an archive file
- `-f` : Specifies the filename of the archive file
- `-z` : Creates a tar file using gzip compression

## Manage package

#### Debian (eg. Ubuntu)

Use `apt-get` to handle packages
```sh
# Synchronize the package index files from their sources again
apt-get update

# Install the latest versions of the packages currently installed
apt-get upgrade

# Install or upgrade packages
apt-get install
apt-get install --no-install-recommends --no-install-suggests -y curl
```

- `--no-install-recommends` : Lets apt-get know not to consider recommended packages as a dependency to install
- `--no-install-suggests` : Lets apt-get know not to consider suggested  packages as a dependency to install
- `-y` : Specifies that apt-get should assume ‘yes’ for all prompts, and should run without any interaction