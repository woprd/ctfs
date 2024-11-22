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
# CTRL-C after couple minutes
```
Then download locally for analysis on local host

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

At this point switched to Python/scapy to explore the packets. 

```python
# main.py
from scapy.all import *

# Load the PCAP file
packets = rdpcap("data.pcap")

dns_q_names = []

# Loop through each packet and isolate DNS packets
for packet in packets:
    # Check if the packet has a DNS layer
    if packet.haslayer(DNS):
        dns_layer = packet.getlayer(DNS)
        
        # Check if it is a DNS query or response
        if dns_layer.qr == 0:  # DNS Query

            qname = dns_layer[DNS].qd.qname.decode()

            dns_q_names.append(qname.replace('.msndhfie.com.',''))

exfil_data = ''.join(dns_q_names)

```

Running that in ipython (could also be a jupyter notebook)

```bash
In [5]: run main.py

In [6]: exfil_data
Out[6]: 'Mwk1Njc4IEtpbmcgV2lsbGlhbSBTdCwgQWRlbGFpZGUsIFNBCTA0MzMgNDU2IDc4OQkkODUsNTAwCTk1MTQ1Njc1MzkwMglIUglydWJ5Lmxld2lzQHNtYXJ0Y2l0eS5jb20uYXUKTGlhbSBXYWxrZXIJMTIzCTQ1NiA3ODkgMTIzCTc4OTAgU3RpcmxpbmcgSHd5LCBQZXJ0aCwgV0EJMDQzNCA1NjcgODkwCSQ3NCwyMDAJNDU2Nzg5MTIzMDEzCU9wZXJhdGlvbnMJbGlhbS53YWxrZXJAc21hcnRjaXR5LmNvbS5hdQpBdmEgTWl0Y2hlbGwJMTI0CTMyMSA2NTQgMTU5CTEyMzQgR2VvcmdlIFN0LCBCcmlzYmFuZSwgUUxECTA0MzUgNjc4IDkwMQkkNzgsMzAwCTMyMTY1NDE1OTEyNAlNYXJrZXRpbmcJYXZhLm1pdGNoZWxsQHNtYXJ0Y2l0eS5jb20uYXUKTm9haCBXb29kCTEyNQk3NTMgMTU5IDQ1Ngk0NTY3IENvbGxpbnMgU3QsIE1lbGJvdXJuZSwgVklDCTA0MzYgNzg5IDAxMgkkODEsOTAwCTc1MzE1OTQ1NjIzNQlTYWxlcwlub2FoLndvb2RAc21hcnRjaXR5LmNvbS5hdQpFbGxhIERhdmlzCTEyNgk5ODcgNDU2IDMyMQk3ODkwIEtpbmcgU3QsIFN5ZG5leSwgTlNXCTA0MzcgODkwIDEyMwkkODcsNjAwCTk4NzQ1NjMyMTM0NglJVAllbGxhLmRhdmlzQHNtYXJ0Y2l0eS5jb20uYXUKQmVuamFtaW4gSGlsbAkxMjcJMTU5IDg1MiA3NTMJMTIzNCBRdWVlbiBTdCwgUGVydGgsIFdBCTA0MzggOTAxIDIzNAkkNzksMjAwCTE1OTg1Mjc1MzQ1NwlGaW5hbmNlCWJlbmphbWluLmhpbGxAc21hcnRjaXR5LmNvbS5hdQpTb3BoaWUgVGhvbXBzb24JMTI4CTQ1NiA5ODcgMzIxCTU2NzggRWR3YXJkIFN0LCBCcmlzYmFuZSwgUUxECTA0MzkgMDEyIDM0NQkkODQsMTAwCTQ1Njk4NzMyMTU2OAlJVAlzb3BoaWUudGhvbXBzb25Ac21hcnRjaXR5LmNvbS5hdQpIYXJyeSBXaWxzb24JMTI5CTk1MSA3NTMgNDU2CTc4OTAgU3dhbnN0b24gU3QsIE1lbGJvdXJuZSwgVklDCTA0NDAgMTIzIDQ1NgkkNzUsODAwCTk1MTc1MzQ1NjY3OQlIUgloYXJyeS53aWxzb25Ac21hcnRjaXR5LmNvbS5hdQpab2UgQ2xhcmtlCTEzMAk4NTIgMTU5IDc1MwkxMjM0IEJhcnJhY2sgU3QsIFBlcnRoLCBXQQkwNDQxIDIzNCA1NjcJJDkwLDUwMAk4NTIxNTk3NTM3ODAJTWFya2V0aW5nCXpvZS5jbGFya2VAc21hcnRjaXR5LmNvbS5hdQpKYWNrIE1hcnRpbgkxMzEJNzUzIDQ1NiA5NTEJNDU2NyBIaW5kbGV5IFN0LCBBZGVsYWlkZSwgU0EJMDQ0MiAzNDUgNjc4CSQ4OCw5MDAJNzUzNDU2OTUxODAxCU9wZXJhdGlvbnMJamFjay5tYXJ0aW5Ac21hcnRjaXR5LmNvbS5hdQpMaWx5IFRheWxvcgkxMzIJMTU5IDc1MyA4NTIJNzg5MCBHZW9yZ2UgU3QsIFN5ZG5leSwgTlNXCTA0NDMgNDU2IDc4OQkkODIsNDAwCTE1OTc1Mzg1MjkwMglTYWxl'
```

---

### Step 3: Flag Retrieval

Adding line to decode from base64 then to UTF-8

```Python
decoded_bytes = base64.b64decode(exfil_data).decode('utf-8')
```

```bash
python main.py 
3679	Sales	james.lee@smartcity.com.au
Isabella Young	120	951 852 753	1234 Little Collins St, Melbourne, VIC	0431 234 567	$87,000	951852753790	Finance	isabella.young@smartcity.com.au
flag{digging_for_dns_data}
Ethan Roberts	121	159 357 951	2345 Bourke St, Sydney, NSW	0432 345 678	$93,000	159357951801	IT	ethan.roberts@smartcity.com.au
Ruby Lewis	122	951 456 753	5678 King William St, Adelaide, SA	0433 456 789	$85,500	951456753902	HR	ruby.lewis@smartcity.com.au
Liam Walker	123	456 789 123	7890 Stirling Hwy, Perth, WA	0434 567 890	$74,200	456789123013	Operations	liam.walker@smartcity.com.au
Ava Mitchell	124	321 654 159	1234 George St, Brisbane, QLD	0435 678 901	$78,300	321654159124	Marketing	ava.mitchell@smartcity.com.au
Noah Wood	125	753 159 456	4567 Collins St, Melbourne, VIC	0436 789 012	$81,900	753159456235	Sales	noah.wood@smartcity.com.au
Ella Davis	126	987 456 321	7890 King St, Sydney, NSW	0437 890 123	$87,600	987456321346	IT	ella.davis@smartcity.com.au
Benjamin Hill	127	159 852 753	1234 Queen St, Perth, WA	0438 901 234	$79,200	159852753457	Finance	benjamin.hill@smartcity.com.au
Sophie Thompson	128	456 987 321	5678 Edward St, Brisbane, QLD	0439 012 345	$84,100	456987321568	IT	sophie.thompson@smartcity.com.au
Harry Wilson	129	951 753 456	78
```


### Voilà Le Flag

**flag{digging_for_dns_data}**


## Solution 2



---

## Notes
- [Any interesting observations or things that helped you solve the challenge]
- [Anything unusual you encountered during the process, such as complex file formats, hidden encryption schemes, or unusual artifacts]
- [Possible improvements or alternative solutions you might have thought about]

---

## Additional Resources (Optional)
- Links to relevant resources, tools, or writeups that helped you in solving the challenge.

---

