---
title: "OverTheWire: Bandit — Write-up (Levels 0–33)"
date: 2023-09-19
draft: false
tags: ["ctf", "overthewire", "bandit", "linux"]
summary: "Full walkthrough of the OverTheWire Bandit wargame."
showToc: true 
---

> **Note on passwords.** OverTheWire's own rules ask players not to post passwords. Every flag in this post is shown as `[REDACTED]` and the private SSH keys are truncated.

## Basic mechanics of this challenge

To start, we connect via SSH with this syntax:

```
ssh bandit[challenge number]@bandit.labs.overthewire.org -p 2220
```

Then we look for the password of the next level. Once we have it, we `exit` the connection and connect again with the obtained credentials.

# LVL 00

**Goal:** Log into the game over SSH and read the first password.
**Key concept:** `ssh` — connecting to a remote host on a custom port.

Let's start by connecting to the host **bandit.labs.overthewire.org** via SSH on port **2220**:

```
$ ssh bandit0@bandit.labs.overthewire.org -p 2220
```

The connection will ask for the password; at this level it is **bandit0**.

Once connected, we can list the contents. The file that stores the next password is the **readme**:

```
bandit0@bandit:~$ ls -la
total 24
drwxr-xr-x  2 root    root    4096 Apr 23 18:04 .
drwxr-xr-x 70 root    root    4096 Apr 23 18:05 ..
-rw-r--r--  1 root    root     220 Jan  6  2022 .bash_logout
-rw-r--r--  1 root    root    3771 Jan  6  2022 .bashrc
-rw-r--r--  1 root    root     807 Jan  6  2022 .profile
-rw-r-----  1 bandit1 bandit0   33 Apr 23 18:04 readme
```

Finally, we `cat` the document and we have the password for the user bandit1:

```
bandit0@bandit:~$ cat readme
[REDACTED]
```

# LVL 01

**Goal:** Read a file whose name is a single dash (`-`).
**Key concept:** referring to an awkward filename by its absolute (or `./`) path so the shell doesn't treat it as an option.

Once connected, we list the current directory with `ls -la` (or `ll`):

```
bandit1@bandit:~$ ls -la
total 24
-rw-r-----  1 bandit2 bandit1   33 Apr 23 18:04 -
drwxr-xr-x  2 root    root    4096 Apr 23 18:04 .
drwxr-xr-x 70 root    root    4096 Apr 23 18:05 ..
-rw-r--r--  1 root    root     220 Jan  6  2022 .bash_logout
-rw-r--r--  1 root    root    3771 Jan  6  2022 .bashrc
-rw-r--r--  1 root    root     807 Jan  6  2022 .profile
```

The password is stored in a document named `-`. This will not allow a normal `cat`. One alternative is to specify the absolute path:

```
bandit1@bandit:~$ cat /home/bandit1/-
[REDACTED]
```

# LVL 02

**Goal:** Read a file whose name contains spaces.
**Key concept:** quoting a filename so the shell treats it as a single argument.

At this level we list and view a file with spaces:

```
bandit2@bandit:~$ ls
spaces in this filename
```

To see the content, put the file name between quotes:

```
bandit2@bandit:~$ cat "spaces in this filename"
[REDACTED]
```

# LVL 03

**Goal:** Find a hidden file inside a directory.
**Key concept:** `ls -la` to reveal dotfiles.

We list and see a directory:

```
bandit3@bandit:~$ ls
inhere
```

Enter it with `cd inhere/` and list, including hidden documents:

```
bandit3@bandit:~/inhere$ ls -la
total 12
drwxr-xr-x 2 root    root    4096 Apr 23 18:04 .
drwxr-xr-x 3 root    root    4096 Apr 23 18:04 ..
-rw-r----- 1 bandit4 bandit3   33 Apr 23 18:04 .hidden
```

We see the `.hidden` file and read it with `cat`:

```
bandit3@bandit:~/inhere$ cat .hidden
[REDACTED]
```

# LVL 04

**Goal:** Identify the only human-readable file among several.
**Key concept:** `file` — determining file type from magic numbers.

List and enter the `inhere` directory. We see 9 files:

```
bandit4@bandit:~$ cd inhere/
bandit4@bandit:~/inhere$ ls
-file00  -file01  -file02  -file03  -file04  -file05  -file06  -file07  -file08  -file09
```

A `cat` on each one would be slow. Instead, we check their types:

```
bandit4@bandit:~/inhere$ file ./-file0*
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: Non-ISO extended-ASCII text, with no line terminators
```

The only readable one is the `ASCII text` file (`-file07`), so we read it:

```
bandit4@bandit:~/inhere$ cat ./-file07
[REDACTED]
```

# LVL 05

**Goal:** Find a file matching a set of properties.
**Key concept:** `find` filtered by size and type.

For this level the password is stored in a file with these properties:

- Human-readable
- 1033 bytes in size
- Not executable
- Inside `inhere`

We can filter by size and type — at this level, that's enough:

```
bandit5@bandit:~$ find ./inhere/ -type f -size 1033c
./inhere/maybehere07/.file2
```

We see the path and `cat` it:

```
bandit5@bandit:~$ cat /home/bandit5/inhere/maybehere07/.file2
[REDACTED]
```

# LVL 06

**Goal:** Find a file on the whole system matching owner, group and size.
**Key concept:** `find` with `-user`/`-group` and redirecting errors with `2>/dev/null`.

The password is in a file with the following properties:

- Owned by user bandit7
- Owned by group bandit6
- 33 bytes in size

```
$ find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
```

We use `-type`, `-user`, `-group` and `-size` to specify the search, and `2>/dev/null` to redirect errors so they aren't shown. Finally, we read the file:

```
bandit6@bandit:~$ cat /var/lib/dpkg/info/bandit7.password
[REDACTED]
```

# LVL 07

**Goal:** Find the password next to a specific word in a large file.
**Key concept:** `grep` to filter lines.

The password is inside `data.txt`, next to the word **millionth**. The file has many lines, so a `cat` piped into `grep` filters the line we care about:

```
bandit7@bandit:~$ cat data.txt | grep millionth
millionth       [REDACTED]
```

# LVL 08

**Goal:** Find the only line that appears once.
**Key concept:** `sort | uniq -u`.

`data.txt` has many repeated strings and only one that is not repeated — that's the password. We sort the output and keep only non-repeated lines:

```
bandit8@bandit:~$ cat data.txt | sort | uniq -u
[REDACTED]
```

# LVL 09

**Goal:** Extract a human-readable string from mostly binary data.
**Key concept:** `strings` + `grep`.

The password is in `data.txt`, in one of the human-readable strings, preceded by several `=` characters:

```
bandit9@bandit:~$ strings data.txt | grep "<mark>"
4</mark>======<mark> the#
</mark>======<mark> password
</mark>======<mark> is
</mark>======== [REDACTED]
```

The `strings` command lists printable character chains, and `grep` filters those containing two or more `=` characters.

# LVL 10

**Goal:** Decode Base64 data.
**Key concept:** `base64 -d`.

`data.txt` contains Base64-encoded data. We `cat` it and pipe it into `base64 -d` (`-d` decodes):

```
bandit10@bandit:~$ cat data.txt | base64 -d
The password is [REDACTED]
```

# LVL 11

**Goal:** Decode a ROT13-rotated string.
**Key concept:** `tr` for character substitution.

The letters (a–z and A–Z) have been rotated by 13 positions:

```
bandit11@bandit:~$ cat data.txt
Gur cnffjbeq vf [ROT13-ENCODED]
```

We can copy the output to [rot13.com](https://rot13.com) to automate the process:

![Pasted image 20230729200841](pasted-image-20230729200841.png)

Or do it in the terminal with `tr`:

```
bandit11@bandit:~$ cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
The password is [REDACTED]
```

With `tr`, the first block `'A-Za-z'` represents all upper- and lower-case letters, and the second block `'N-ZA-Mn-za-m'` represents the 13-position shift for each letter.

# LVL 12

**Goal:** Recover a file that has been hexdumped and compressed several times.
**Key concept:** `xxd -r` to reverse a hexdump, then repeated decompression with `7z`.

`data.txt` is a hexdump of a file that has been compressed several times.

![Pasted image 20230729215147](pasted-image-20230729215147.png)

We start by copying the content of `data.txt` to a file on our main server:

```
nano data.txt
```

![Pasted image 20230729221110](pasted-image-20230729221110.png)

```
└─$ cat data.txt | xxd -r | sponge data.txt
```

With `xxd -r` we reverse the hexadecimal file, and with `sponge` we write the result back to the file:

```
└─$ file data.txt
data.txt: gzip compressed data, was "data2.bin", last modified: Sun Apr 23 18:04:23 2023, max compression, from Unix, original size modulo 2^32 581
```

Running `file` shows it's now a `gzip` compressed file. To decompress, we use `7z`, which handles all sorts of file types. First, the `l` parameter lists the contents (here, another compressed file):

```
└─$ 7z l data.txt

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21

Listing archive: data.txt
--
Path = data.txt
Type = gzip
Headers Size = 20

   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2023-04-23 14:04:23 .....          581          614  data2.bin
------------------- ----- ------------ ------------  ------------------------
2023-04-23 14:04:23                581          614  1 files
```

Then the `x` parameter extracts it:

```
└─$ 7z x data.txt

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21

Extracting archive: data.txt
--
Path = data.txt
Type = gzip
Headers Size = 20

Everything is Ok

Size:       581
Compressed: 614
```

Once unzipped, a new document is created and we have to unzip it again:

```
└─$ ls
data2      data5.bin  data8.bin
data2.bin  data6      data9.bin
data4.bin  data6.bin  data.txt

└─$ file data9.bin
data9.bin: ASCII text
```

The process is repeated ~9 times, checking `file [file name]` after each step, until we reach a file that returns `ASCII text`:

```
└─$ cat data9.bin
The password is [REDACTED]
```

> To make this task easier, you can use this [program](https://github.com/Masiouslow/7z-unzip-automator) to automate the process.

# LVL 13

**Goal:** Use a private SSH key instead of a password to log into the next level.
**Key concept:** `ssh -i` with a private key.

The password is in `/etc/bandit_pass/bandit14` and can only be read by user `bandit14`. This time we don't log in with a password — we log in with a private SSH key:

```
bandit13@bandit:~$ cat sshkey.private
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAxkkOE83W2cOT7IWhFc9aPaaQmQDdgzuXCv+ppZHa++buSkN+
[... private key body redacted — read it yourself on the server ...]
-----END RSA PRIVATE KEY-----
```

It's a private SSH key, which we'll use to connect without a password:

```
ssh -i sshkey.private bandit14@localhost -p 2220
```

We open the connection with `ssh` and `-i` to specify the private key, plus `-p` for the port:

```
The authenticity of host '[localhost]:2220 ([127.0.0.1]:2220)' can't be established.
ED25519 key fingerprint is SHA256:C2ihUBV7ihnV1wUXRb4RrEcLfXC5CXlhmAAM/urerLY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

We answer `yes`. After this we're in bandit14, and we `cat` the password:

```
bandit14@bandit:~/.ssh$ cat /etc/bandit_pass/bandit14
[REDACTED]
```

# LVL 14

**Goal:** Submit the current password to a local service to get the next one.
**Key concept:** `nc` (netcat) to talk to a TCP port.

We simply use `nc` to connect to localhost on **port 30000** and provide the level password. It returns the next password directly:

```
bandit14@bandit:~$ nc localhost 30000
[REDACTED]
Correct!
[REDACTED]
```

# LVL 15

**Goal:** Submit the password over an SSL/TLS connection.
**Key concept:** `ncat --ssl`.

We connect via localhost using SSL encryption on port **30001**. We use `ncat` with `--ssl`, followed by the localhost IP and the port:

```
bandit15@bandit:~$ ncat --ssl 127.0.0.1 30001
[REDACTED]
Correct!
[REDACTED]
```

# LVL 16

**Goal:** Find which of several ports speaks SSL and returns the password.
**Key concept:** `nmap` for port scanning, then `ncat --ssl`.

We scan the open ports between 31000 and 32000; one of them returns the password over an encrypted connection:

```
bandit16@bandit:~$ nmap --open -p31000-32000 127.0.0.1

Nmap scan report for localhost (127.0.0.1)
PORT      STATE SERVICE
31046/tcp open  unknown
31518/tcp open  unknown
31691/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown
```

We use `nmap` with `--open` to show only open ports, `-p31000-32000` for the range, and the localhost IP. Then we test the ports with `ncat --ssl` (entering the level password when connecting):

```
bandit16@bandit:~$ ncat --ssl 127.0.0.1 31790
[REDACTED]
Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
[... private key body redacted — read it yourself on the server ...]
-----END RSA PRIVATE KEY-----
```

Once we have the RSA private key, we create a temporary directory in `/tmp` and store the key there:

```
bandit16@bandit:~$ cd /tmp/
bandit16@bandit:/tmp$ mktemp -d
/tmp/tmp.khzgBAa4Mx
```

```
bandit16@bandit:/tmp/tmp.Cv7pQBBWcW$ nano id_rsa
```

![Pasted image 20230818154217](pasted-image-20230818154217.png)

```
bandit16@bandit:/tmp/tmp.Cv7pQBBWcW$ ls
id_rsa
bandit16@bandit:/tmp/tmp.Cv7pQBBWcW$ chmod 600 id_rsa
```

With `chmod 600` we assign the permissions SSH requires for a private key. Finally we connect and read the next password:

```
bandit16@bandit:/tmp/tmp.Cv7pQBBWcW$ ssh -i id_rsa bandit17@localhost -p 2220
```

```
bandit17@bandit:~$ cat /etc/bandit_pass/bandit17
[REDACTED]
```

# LVL 17

**Goal:** Find the single changed line between two files.
**Key concept:** `diff`.

We have two files with almost identical contents; one line was modified and holds the password:

```
bandit17@bandit:~$ ls
passwords.new  passwords.old
```

The password is in **passwords.new**. With `diff` we see the difference between `.old` and `.new`:

```
bandit17@bandit:~$ diff passwords.old passwords.new
42c42
< [REDACTED-OLD]
---
> [REDACTED]
```

# LVL 18

**Goal:** Read a file even though `.bashrc` closes the session on login.
**Key concept:** running a command directly through SSH, or forcing a non-interactive shell.

The password is stored in a file **readme**, but logging in shows a "connection closed" message:

```
Byebye !
Connection to bandit.labs.overthewire.org closed.
```

We can slip the `cat` command into the SSH request:

```
└─$ ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
[REDACTED]
```

Or force a shell to open:

```
└─$ ssh bandit18@bandit.labs.overthewire.org -p 2220 bash

ls
readme
cat readme
[REDACTED]
```

# LVL 19

**Goal:** Use a SUID helper binary to read a protected file.
**Key concept:** SUID binaries run as their owner.

We have a file with the SUID bit set:

```
bandit19@bandit:~$ ls -la bandit20-do
-rwsr-x--- 1 bandit20 bandit19 14876 Apr 23 18:04 bandit20-do
```

These files let us run commands as the file's owner. We can pass `cat` directly:

```
bandit19@bandit:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
[REDACTED]
```

Or force a shell with `bash -p`:

```
bandit19@bandit:~$ ./bandit20-do bash -p
bash-5.1$ whoami
bandit20
bash-5.1$ cat /etc/bandit_pass/bandit20
[REDACTED]
```

# LVL 20

**Goal:** Feed a password to a binary that connects back to a port you control.
**Key concept:** setting up a listener with `nc -nlvp` and using two terminals.

We find a binary that connects to localhost on a port we specify; if we send it the level 20 password over that connection, it returns the level 21 password.

![Pasted image 20230818205422](pasted-image-20230818205422.png)

First, open two terminals, both connected as bandit20. In the first, listen on a port of our choice (here 4646):

```
bandit20@bandit:~$ nc -nlvp 4646
Listening on 0.0.0.0 4646
```

In the second, run the binary pointing at that port:

```
bandit20@bandit:~$ ./suconnect 4646
```

Netcat reports a new connection; from that same terminal we type the previous level's password:

```
bandit20@bandit:~$ nc -nlvp 4646
Listening on 0.0.0.0 4646
Connection received on 127.0.0.1 59652
[REDACTED]
```

![Pasted image 20230818211113](pasted-image-20230818211113.png)

This returns the password for bandit21: `[REDACTED]`.

# LVL 21

**Goal:** Read a password written to `/tmp` by a cron job.
**Key concept:** inspecting cron jobs in `/etc/cron.d`.

We analyze the cron tasks and take advantage of them:

```
bandit21@bandit:~$ cd /etc/cron.d
```

![Pasted image 20230818212906](pasted-image-20230818212906.png)

In `/etc/cron.d` there are several cron tasks:

```
bandit21@bandit:/etc/cron.d$ cat cronjob_bandit22
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```

A `.sh` script runs at regular intervals. Let's read it:

```
bandit21@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit22.sh
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

The script writes the bandit22 password to a file in `/tmp`. We just read it:

```
bandit21@bandit:/etc/cron.d$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
[REDACTED]
```

# LVL 22

**Goal:** Predict the (dynamically named) file a cron job writes to.
**Key concept:** reading a script to reproduce how it derives a filename.

Again we take advantage of a cron task. We enter `/etc/cron.d/` and read `cronjob_bandit23`:

![Pasted image 20230907161206](pasted-image-20230907161206.png)

```
@reboot bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
```

The task runs `cronjob_bandit23.sh`, so we analyze it:

```
bandit22@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit23.sh
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

The script saves the next level's password to a file in `/tmp` named after the user's md5sum: `echo I am user $myname | md5sum | cut -d ' ' -f 1`. We run that line as bandit23 to find the filename:

```
bandit22@bandit:~$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
8ca319486bfbbc3663ea0fbe81326349
```

It's copying it to `/tmp/8ca319486bfbbc3663ea0fbe81326349`. We read it:

```
bandit22@bandit:~$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
[REDACTED]
```

# LVL 23

**Goal:** Get a cron job (running as bandit24) to execute a script we own.
**Key concept:** writing an exploit script into a "drop folder" that a privileged cron job runs.

This time we create a script to take advantage of a cron job:

```
bandit23@bandit:~$ cd /etc/cron.d
bandit23@bandit:/etc/cron.d$ cat cronjob_bandit23
@reboot bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
```

Let's see what the cron job runs — the program `/usr/bin/cronjob_bandit24.sh`:

```
bandit23@bandit:~$ cat /usr/bin/cronjob_bandit24.sh
#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname/foo || exit 1
echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "%U" ./$i)"
        if [ "${owner}" = "bandit23" ]; then
            timeout -s 9 60 ./$i
        fi
        rm -rf ./$i
    fi
done
```

It executes and then deletes all files in `/var/spool/$myname/foo` at regular intervals. We create a temporary working directory and a script:

```
bandit23@bandit:/var/spool$ mktemp -d
/tmp/tmp.aqReHiZ3M4
bandit23@bandit:/var/spool$ cd /tmp/tmp.aqReHiZ3M4
bandit23@bandit:/tmp/tmp.aqReHiZ3M4$ nano test.sh
```

The script:

```bash
#!/bin/bash

cat /etc/bandit_pass/bandit24 > /tmp/skWEwN9saS/pass.log
# the output directory must be the temporary directory you have created

chmod o+r /tmp/skWEwN9saS/pass.log
# the file is created by bandit24, so we give read access to "other"
```

Assign the necessary permissions to both the script and the directory:

```
bandit23@bandit:/tmp/tmp.skWEwN9saS$ chmod +x test.sh
bandit23@bandit:/tmp/tmp.skWEwN9saS$ chmod o+wx /tmp/tmp.skWEwN9saS/
```

Copy the script into the drop folder and monitor with `watch`:

```
bandit23@bandit:/tmp/tmp.skWEwN9saS$ cp test.sh /var/spool/bandit24/foo/testing
bandit23@bandit:/tmp/tmp.skWEwN9saS$ watch -n 1 ls -l
```

![Pasted image 20230908003822](pasted-image-20230908003822.png)

Wait for the file to appear, then read it:

```
bandit23@bandit:/tmp/tmp.skWEwN9saS$ cat pass.log
[REDACTED]
```

# LVL 24

**Goal:** Brute-force a 4-digit PIN against a daemon.
**Key concept:** generating a wordlist in bash and piping it into `nc`.

We must brute-force a daemon to return the next password. Create a temporary directory:

```
bandit24@bandit:~$ mktemp -d
/tmp/tmp.KN9zXZurDx
bandit24@bandit:~$ cd /tmp/tmp.KN9zXZurDx
```

Create a file with every PIN between 0 and 10000, each prefixed with the previous level's password:

```
bandit24@bandit:/tmp/tmp.KN9zXZurDx$ for pincode in {0..10000}; do echo "[BANDIT24-PASSWORD]" $pincode; done > PinList.txt
```

Feed the list into `nc` on port 30002 and filter out the "Wrong"/"Please" responses to see only the correct one:

```
bandit24@bandit:/tmp/tmp.MxkQ0O8MwY$ tac PinList.txt | nc localhost 30002 | grep -vE "Wrong|Please"
Correct!
The password of user bandit25 is [REDACTED]
```

# LVL 25

**Goal:** Escape a restricted pager into a shell.
**Key concept:** `more` (via `vi`) can spawn a shell when the terminal is too small to page.

We'll abuse the `more` pager in the next level.

![Pasted image 20230911014429](pasted-image-20230911014429.png)

The connection normally closes immediately. To keep it open, we shrink the terminal so `more` is forced into paging mode (as in the photo above), then connect:

```
bandit25@bandit:~$ ssh -i bandit26.sshkey bandit26@localhost -p 2220
```

![Pasted image 20230911014713](pasted-image-20230911014713.png)

Make sure you're at the `--More--` prompt; you can restore the original terminal size now.

Press `v` to enter `vi`, then declare:

```
:set shell=/bin/bash
```

![Pasted image 20230911015016](pasted-image-20230911015016.png)

Then call a shell:

```
:shell
```

![Pasted image 20230915192427](pasted-image-20230915192427.png)
![Pasted image 20230915192535](pasted-image-20230915192535.png)

We are now bandit26:

```
bandit26@bandit:~$ cat /etc/bandit_pass/bandit26
[REDACTED]
```

We can see the password, but it won't help us directly — without leaving bandit26, we move on to the next level.

# LVL 26

**Goal:** Use a SUID helper to read the next password.
**Key concept:** SUID binaries again — passing a command to run as the owner.

We take advantage of a SUID binary:

```
bandit26@bandit:~$ ls -la
-rwsr-x---  1 bandit27 bandit26 14876 Apr 23 18:04 bandit27-do
```

Since `bandit27-do` is SUID, we can pass it a command:

```
bandit26@bandit:~$ ./bandit27-do cat /etc/bandit_pass/bandit27
[REDACTED]
```

# LVL 27

**Goal:** Clone a git repo over SSH and read the README.
**Key concept:** `git clone ssh://…` with a custom port.

We start working with git. Create and enter a temporary directory:

```
bandit27@bandit:~$ mktemp -d
/tmp/tmp.P9OK4nMR0X
bandit27@bandit:~$ cd /tmp/tmp.P9OK4nMR0X
```

Clone the repo, specifying port 2220:

```
git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/
```

```
ED25519 key fingerprint is SHA256:C2ihUBV7ihnV1wUXRb4RrEcLfXC5CXlhmAAM/urerLY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

We answer `yes` and provide this level's password:

```
bandit27-git@localhost's password:
remote: Enumerating objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (3/3), done.
```

Enter the repository and read the README:

```
bandit27@bandit:/tmp/tmp.RxeSDV8KOx$ cd repo/
bandit27@bandit:/tmp/tmp.RxeSDV8KOx/repo$ ls
README
bandit27@bandit:/tmp/tmp.RxeSDV8KOx/repo$ cat README
The password to the next level is: [REDACTED]
```

# LVL 28

**Goal:** Recover a password removed in a later commit.
**Key concept:** `git log` and `git show` to inspect history.

Same mechanics. Create a temporary directory:

```
bandit28@bandit:~$ mktemp -d
/tmp/tmp.fekcDAMvX6
bandit28@bandit:~$ cd /tmp/tmp.fekcDAMvX6
```

Clone the repo:

```
git clone ssh://bandit28-git@localhost:2220/home/bandit28-git/repo
```

```
bandit28-git@localhost's password:
remote: Enumerating objects: 9, done.
Receiving objects: 100% (9/9), done.
Resolving deltas: 100% (2/2), done.
```

With `git log` we see the commits; the interesting one is subject `fix info leak`:

```
bandit28@bandit:/tmp/tmp.fekcDAMvX6/repo$ git log

commit 899ba88df296331cc01f30d022c006775d467f28 (HEAD -> master, origin/master, origin/HEAD)
Author: Morla Porla <morla@overthewire.org>
Date:   Sun Apr 23 18:04:39 2023 +0000

    fix info leak

commit abcff758fa6343a0d002a1c0add1ad8c71b88534
Author: Morla Porla <morla@overthewire.org>
Date:   Sun Apr 23 18:04:39 2023 +0000

    add missing data

commit c0a8c3cf093fba65f4ee0e1fe2a530b799508c78
Author: Ben Dover <noone@overthewire.org>
Date:   Sun Apr 23 18:04:39 2023 +0000

    initial commit of README.md
```

Show that commit with `git show [identifier]`:

```
bandit28@bandit:/tmp/tmp.fekcDAMvX6/repo$ git show 899ba88df296331cc01f30d022c006775d467f28

    fix info leak

diff --git a/README.md b/README.md
--- a/README.md
+++ b/README.md
@@ -4,5 +4,5 @@ Some notes for level29 of bandit.
 ## credentials

 - username: bandit29
-- password: [REDACTED]
+- password: xxxxxxxxxx
```

The password removed in that commit is the one we need.

# LVL 29

**Goal:** Find the password on a different git branch.
**Key concept:** `git checkout` / branches.

Create and enter a temporary directory, then clone:

```
bandit29@bandit:~$ mktemp -d
/tmp/tmp.qDgJnQeiO0
bandit29@bandit:~$ cd /tmp/tmp.qDgJnQeiO0
```

```
bandit29@bandit:/tmp/tmp.qDgJnQeiO0$ git clone ssh://bandit29-git@localhost:2220/home/bandit29-git/repo
```

Enter the repo, switch to the `dev` branch and read `README.md`:

```
bandit29@bandit:/tmp/tmp.qDgJnQeiO0/repo$ git checkout dev
Already on 'dev'
Your branch is up to date with 'origin/dev'.

bandit29@bandit:/tmp/tmp.qDgJnQeiO0/repo$ cat README.md
# Bandit Notes
Some notes for bandit30 of bandit.

## credentials

- username: bandit30
- password: [REDACTED]
```

# LVL 30

**Goal:** Reveal a password hidden in a git tag.
**Key concept:** `git tag` / `git show <tag>`.

Create a temporary directory and clone the repo:

```
bandit30@bandit:~$ mktemp -d
/tmp/tmp.85L46g5uSl
bandit30@bandit:~$ cd /tmp/tmp.85L46g5uSl
```

```
bandit30@bandit:/tmp/tmp.85L46g5uSl$ git clone ssh://bandit30-git@localhost:2220/home/bandit30-git/repo
```

List the tags and show the `secret` one:

```
bandit30@bandit:/tmp/tmp.HQrotoWMqm/repo$ git tag
secret
bandit30@bandit:/tmp/tmp.HQrotoWMqm/repo$ git show secret
[REDACTED]
```

# LVL 31

**Goal:** Push a commit that satisfies a server-side check.
**Key concept:** `git add -f`, `git commit`, `git push`.

Create a temporary directory and clone the repo:

```
bandit31@bandit:~$ mktemp -d
/tmp/tmp.PA8cRNfNcO
bandit31@bandit:~$ cd /tmp/tmp.PA8cRNfNcO
```

```
bandit31@bandit:/tmp/tmp.PA8cRNfNcO$ git clone ssh://bandit31-git@localhost:2220/home/bandit31-git/repo
```

Create a file `key.txt` with the content `May I come in?`:

```
nano key.txt
```

![Pasted image 20230919195210](pasted-image-20230919195210.png)

Prepare the commit (note the `-f` to force-add the ignored file):

```
bandit31@bandit:/tmp/tmp.qURr3NtElF/repo$ git add -f key.txt
bandit31@bandit:/tmp/tmp.qURr3NtElF/repo$ git commit -m "Creating a key"

[master d4d54a9] Creating a key
 1 file changed, 1 insertion(+), 1 deletion(-)
```

Finally, push it and provide the password:

```
bandit31@bandit:/tmp/tmp.qURr3NtElF/repo$ git push -u origin master

remote: ### Attempting to validate files... ####
remote:
remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
remote:
remote: Well done! Here is the password for the next level:
remote: [REDACTED]
remote:
remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
```

# LVL 32

**Goal:** Escape a shell that upper-cases every command.
**Key concept:** `$0` expands to the shell's own name, bypassing the upper-casing.

The final level drops us into a shell that upper-cases input. We break out with `$0`:

```
WELCOME TO THE UPPERCASE SHELL
>> $0
$
```

Now in a normal shell, we go to bandit33's home and read the README:

```
$ cd /home/bandit33
$ ls
README.txt
$ cat README.txt
Congratulations on solving the last level of this game!

At this moment, there are no more levels to play in this game. However, we are constantly working
on new levels and will most likely expand this game with more levels soon.
Keep an eye out for an announcement on our usual communication channels!
In the meantime, you could play some of our other wargames.

If you have an idea for an awesome new level, please let us know!
$
```

# END
