# CTF Writeup: [Challenge Name]

## Table of Contents
1. [Challenge Overview](#challenge-overview)
2. [Tools Used](#tools-used)
3. [Solution Steps](#solution-steps)
    - [Step 1: Initial Analysis](#step-1-initial-analysis)
    - [Step 2: Investigation/Exploitation](#step-2-investigationexploitation)
    - [Step 3: Flag Retrieval](#step-3-flag-retrieval)
4. [Flag](#flag)
5. [Notes](#notes)

---

## Challenge Overview
* **Category:** Digital Forensics
* **Difficulty:** [Medium]
* **Description:**  
 We have access to network tap between server and gateway accessible via ssh. Need to determine how data exfiltrated and flag is contained in the same. 


---

Posting 2 solutions here: Solution 1 was during the comp., Solution 2 was thought of with hindsight. 

## Tools Used

### Solution 1

- [Wireshark]([Link to tool, if applicable](https://www.wireshark.org/))
- [Python]

### Solution 2

- [Suricata](https://suricata.io/)
- [jq](https://jqlang.github.io/jq/)

---

## Solution 1 and 2

We were provided ssh creds so first step was to ssh to the tap and capture some packets 

```bash
ssh <user>@<ip>
sudo tcpdump -i eth0 -w data.pcap
```
Then copy locally for analysis

```bash
scp <user>@<ip>:<path>/data.pcap .
```

## Solution 1

### Step 1: Initial Analysis

Normally I would jump straight to Python and `scapy` to explore packets as I loathe GUIs. A teammate was wireshark-savvy so we buddied up. 

After first checking Statistics > Conversations, a cursory scroll through saw something odd in the DNS packet in light blue here:

![BitByBit1](https://github.com/user-attachments/assets/ee8311df-1d3e-40a2-9e75-ab432570428d)

Looks like a base64 encoded string? 

---

### Step 2: Investigation/Exploitation
In this section, describe how you investigated the challenge and performed exploitation, which may vary based on the challenge type:

#### For **Steganography** Challenges:
- Investigate the file for hidden data in the image, audio, or text.
- Tools such as `stegsolve`, `zsteg`, `binwalk`, or custom scripts might be used to extract hidden information.

**Example:**  
- I used `zsteg` to check for hidden data in the image. This revealed a hidden message embedded in the least significant bits of the pixels.

#### For **Digital Forensics** Challenges:
- Analyze any provided disk images, logs, or memory dumps.
- Use tools like `Autopsy`, `Volatility`, `Wireshark`, or even custom scripts to investigate artifacts, timestamps, or file signatures.

**Example:**  
- The disk image appeared to contain deleted files. Using `foremost` and `Autopsy`, I was able to recover a deleted file that contained important metadata.

#### For **Crypto** Challenges:
- Try common attacks or algorithms to break the encryption.
- Use Python libraries, `hashcat`, or `John the Ripper` to crack hashes.

**Example:**  
- The challenge used a Caesar cipher. After testing various shifts, I was able to decrypt the flag.

#### For **Web** Challenges:
- Analyze the source code or web requests (SQL injections, XSS, etc.).
- Tools like `Burp Suite` or `sqlmap` may assist in finding vulnerabilities.

**Example:**  
- I identified a SQL injection vulnerability in the login form and extracted the flag from the database.

---

### Step 3: Flag Retrieval
In this section, describe how you successfully retrieved the flag. This might include decoding messages, cracking passwords, or extracting hidden data.

**Example:**  
- After extracting hidden text from the image, I found the flag in the format `CTF{hidden_message_here}`.

---

## Flag

---

## Notes
- [Any interesting observations or things that helped you solve the challenge]
- [Anything unusual you encountered during the process, such as complex file formats, hidden encryption schemes, or unusual artifacts]
- [Possible improvements or alternative solutions you might have thought about]

---

## Additional Resources (Optional)
- Links to relevant resources, tools, or writeups that helped you in solving the challenge.

---

