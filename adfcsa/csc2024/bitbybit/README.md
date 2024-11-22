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
* **Description:** We have access to network tap between server and gateway accessible via ssh. Need to determine how data was exfiltrated and the flag is contained in that data. 

Posting 2 solutions here: Solution 1 was during the comp.; Solution 2 was an after-thought. 

## Tools Used

### Solution 1

- [Wireshark](https://www.wireshark.org/)
- Python/iPython

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

Then downloaded the capture to local host for analysis
```bash
scp <user>@<ip>:<path>/data.pcap .
```

## Solution 1

### Step 1: Initial Analysis

Normally I would jump straight to Python and the `scapy` package to explore packets as I have old timer's aversion to GUIs (back in my day it was all mainframes and punch-cards). 

A teammate was wireshark-savvy and kindly schooled the boomer. So we opened the `data.pcap` file in Wireshark.

After first checking `Statistics > Conversations` with no joy, a cursory scroll through the packets saw something odd in the DNS packet in light blue here:

![BitByBit1](https://github.com/user-attachments/assets/ee8311df-1d3e-40a2-9e75-ab432570428d)

Looks like a base64 encoded string in DNS traffic to me. 

---

### Step 2: Detailed Analysis

At this point we switched to Python/scapy to explore the packets. 

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

            # append to big string, assumption here is the attacker is exfiltraing data piecemeal in FQDNs of DNS queries
            dns_q_names.append(qname.replace('.msndhfie.com.',''))

exfil_data = ''.join(dns_q_names)

```

Running that in ipython (could also be a jupyter notebook)

```bash
In [5]: run main.py

In [6]: exfil_data
Out[6]: 'Mwk1Njc4IEtpbmcgV2lsbGlhbSBTdCwgQWRlbGFpZGUsIFNBCTA0MzMgNDU2IDc4OQkkODUsNTAwCTk1MTQ1Njc1MzkwMglIUglydWJ5Lmxld2lzQHNtYXJ0Y2l0eS5jb20uYXUKTGlhbSBXYWxrZXIJMTIzCTQ1NiA3ODkgMTIzCTc4OTAgU3RpcmxpbmcgSHd5LCBQZXJ0aCwgV0EJMDQzNCA1NjcgODkwCSQ3NCwyMDAJNDU2Nzg5MTIzMDEzCU9wZXJhdGlvbnMJbGlhbS53YWxrZXJAc21hcnRjaXR5LmNvbS5hdQpBdmEgTWl0Y2hlbGwJMTI0CTMyMSA2NTQgMTU5CTEyMzQgR2VvcmdlIFN0LCBCcmlzYmFuZSwgUUxECTA0MzUgNjc4IDkwMQkkNzgsMzAwCTMyMTY1NDE1OTEyNAlNYXJrZXRpbmcJYXZhLm1pdGNoZWxsQHNtYXJ0Y2l0eS5jb20uYXUKTm9haCBXb29kCTEyNQk3NTMgMTU5IDQ1Ngk0NTY3IENvbGxpbnMgU3QsIE1lbGJvdXJuZSwgVklDCTA0MzYgNzg5IDAxMgkkODEsOTAwCTc1MzE1OTQ1NjIzNQlTYWxlcwlub2FoLndvb2RAc21hcnRjaXR5LmNvbS5hdQpFbGxhIERhdmlzCTEyNgk5ODcgNDU2IDMyMQk3ODkwIEtpbmcgU3QsIFN5ZG5leSwgTlNXCTA0MzcgODkwIDEyMwkkODcsNjAwCTk4NzQ1NjMyMTM0NglJVAllbGxhLmRhdmlzQHNtYXJ0Y2l0eS5jb20uYXUKQmVuamFtaW4gSGlsbAkxMjcJMTU5IDg1MiA3NTMJMTIzNCBRdWVlbiBTdCwgUGVydGgsIFdBCTA0MzggOTAxIDIzNAkkNzksMjAwCTE1OTg1Mjc1MzQ1NwlGaW5hbmNlCWJlbmphbWluLmhpbGxAc21hcnRjaXR5LmNvbS5hdQpTb3BoaWUgVGhvbXBzb24JMTI4CTQ1NiA5ODcgMzIxCTU2NzggRWR3YXJkIFN0LCBCcmlzYmFuZSwgUUxECTA0MzkgMDEyIDM0NQkkODQsMTAwCTQ1Njk4NzMyMTU2OAlJVAlzb3BoaWUudGhvbXBzb25Ac21hcnRjaXR5LmNvbS5hdQpIYXJyeSBXaWxzb24JMTI5CTk1MSA3NTMgNDU2CTc4OTAgU3dhbnN0b24gU3QsIE1lbGJvdXJuZSwgVklDCTA0NDAgMTIzIDQ1NgkkNzUsODAwCTk1MTc1MzQ1NjY3OQlIUgloYXJyeS53aWxzb25Ac21hcnRjaXR5LmNvbS5hdQpab2UgQ2xhcmtlCTEzMAk4NTIgMTU5IDc1MwkxMjM0IEJhcnJhY2sgU3QsIFBlcnRoLCBXQQkwNDQxIDIzNCA1NjcJJDkwLDUwMAk4NTIxNTk3NTM3ODAJTWFya2V0aW5nCXpvZS5jbGFya2VAc21hcnRjaXR5LmNvbS5hdQpKYWNrIE1hcnRpbgkxMzEJNzUzIDQ1NiA5NTEJNDU2NyBIaW5kbGV5IFN0LCBBZGVsYWlkZSwgU0EJMDQ0MiAzNDUgNjc4CSQ4OCw5MDAJNzUzNDU2OTUxODAxCU9wZXJhdGlvbnMJamFjay5tYXJ0aW5Ac21hcnRjaXR5LmNvbS5hdQpMaWx5IFRheWxvcgkxMzIJMTU5IDc1MyA4NTIJNzg5MCBHZW9yZ2UgU3QsIFN5ZG5leSwgTlNXCTA0NDMgNDU2IDc4OQkkODIsNDAwCTE1OTc1Mzg1MjkwMglTYWxl'
```

Let's decode the string.

Adding line to `main.py` to decode from base64 then to UTF-8 and print that.

```Python
decoded_bytes = base64.b64decode(exfil_data).decode('utf-8')
print(decoded_bytes)
```

---

### Step 3: Flag Retrieval

Re-ran the script

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

This one is much shorter as I since learned that suricata (even with default rules) can analyse pcap files and log alerts on unusual network traffic. 

```bash
suricata -r data.pcap 
```

Inspect the output

```bash
head -n5 eve.json
{"timestamp":"2024-11-13T12:22:00.381225+1100","flow_id":229978107844536,"pcap_cnt":15,"event_type":"dns","src_ip":"172.18.0.2","src_port":41195,"dest_ip":"92.42.1.83","dest_port":53,"proto":"UDP","pkt_src":"wire/pcap","dns":{"type":"query","id":4908,"rrname":"bQo=.msndhfie.com","rrtype":"TXT","tx_id":0,"opcode":0}}
{"timestamp":"2024-11-13T12:21:43.564905+1100","flow_id":252156359044262,"event_type":"flow","src_ip":"172.18.0.2","src_port":52526,"dest_ip":"23.221.133.223","dest_port":443,"proto":"TCP","flow":{"pkts_toserver":9,"pkts_toclient":7,"bytes_toserver":1369,"bytes_toclient":5372,"start":"2024-11-13T12:22:00.386389+1100","end":"2024-11-13T12:22:00.395461+1100","age":0,"state":"new","reason":"timeout","alerted":false},"tcp":{"tcp_flags":"00","tcp_flags_ts":"00","tcp_flags_tc":"00"}}
{"timestamp":"2024-11-13T12:23:10.454889+1100","flow_id":1953734273435255,"pcap_cnt":38,"event_type":"dns","src_ip":"172.18.0.2","src_port":48963,"dest_ip":"92.42.1.83","dest_port":53,"proto":"UDP","pkt_src":"wire/pcap","dns":{"type":"query","id":15154,"rrname":"TmFtZQlFbXBsb3llZSBJRAlUYXggRmlsZSBOdW1iZXIgKFRGTikJQWRkcmVz.msndhfie.com","rrtype":"TXT","tx_id":0,"opcode":0}}
{"timestamp":"2024-11-13T12:21:43.564905+1100","flow_id":1036130223887667,"event_type":"flow","src_ip":"104.244.42.65","src_port":443,"dest_ip":"172.18.0.2","dest_port":33456,"proto":"TCP","flow":{"pkts_toserver":2,"pkts_toclient":1,"bytes_toserver":156,"bytes_toclient":66,"start":"2024-11-13T12:21:55.306778+1100","end":"2024-11-13T12:21:55.346794+1100","age":0,"state":"new","reason":"timeout","alerted":false},"tcp":{"tcp_flags":"00","tcp_flags_ts":"00","tcp_flags_tc":"00"}}
{"timestamp":"2024-11-13T12:23:12.218431+1100","flow_id":93729663661043,"pcap_cnt":84,"event_type":"dns","src_ip":"172.18.0.2","src_port":33672,"dest_ip":"92.42.1.83","dest_port":53,"proto":"UDP","pkt_src":"wire/pcap","dns":{"type":"query","id":4910,"rrname":"cwlQaG9uZSBOdW1iZXIJU2FsYXJ5CUJhbmsgQWNjb3VudCBOdW1iZXIJRGVw.msndhfie.com","rrtype":"TXT","tx_id":0,"opcode":0}}
```

I then asked a trusty LLM to create a chained jq command to implement the same logic as we did for python. 

```bash
jq -r 'select(.event_type == "dns") | .dns.rrname' eve.json | sed 's/\.msndhfie\.com//g' | tr -d '\n' | base64 --decode
Name    Employee ID     Tax File Number (TFN)   Address Phone Number    Salary  Bank Account Number      Department      Email
John Doe        101     123 456 789     1234 George St, Sydney, NSW     0412 345 678    $85,000  123456789012    IT      john.doe@smartcity.com.au
Jane Smith      102     987 654 321     5678 Collins St, Melbourne, VIC 0423 456 789    $92,000  987654321098    HR      jane.smith@smartcity.com.au
Bob Johnson     103     135 792 468     2468 Queen St, Brisbane, QLD    0434 567 890    $75,000  135792468012    Finance bob.johnson@smartcity.com.au
Alice Brown     104     246 810 135     7890 Flinders St, Adelaide, SA  0415 678 901    $88,500  246810135024    IT      alice.brown@smartcity.com.au
Michael Green   105     975 312 468     1234 King William St, Perth, WA 0416 789 012    $67,500  975312468135    Operations      michael.green@smartcity.com.au
Sarah White     106     321 654 987     4567 Swanston St, Melbourne, VIC        0417 890 123     $82,300 321654987246    HR      sarah.white@smartcity.com.au
David Black     107     789 456 123     8901 Bourke St, Sydney, NSW     0418 901 234    $69,000  789456123357    Sales   david.black@smartcity.com.au
Emily Harris    108     159 753 246     2345 Edward St, Brisbane, QLD   0419 012 345    $94,200  159753246468    Finance emily.harris@smartcity.com.au
Daniel Cooper   109     654 321 987     5678 Pirie St, Adelaide, SA     0420 123 456    $77,400  654321987579    IT      daniel.cooper@smartcity.com.au
Laura King      110     963 258 741     7890 Stirling Hwy, Perth, WA    0421 king@smartcity.com.au
Tom Wright      111     852 147 963     1234 Hay St, Sydney, NSW        0422 345 678    $76234 567       $83,900 963258741680    Marketing       laura.,800      852147963791    Operations       tom.wright@smartcity.com.au
Olivia Scott    112     951 753 852     4567 Barrack St, Perth, WA      0423 456 789    $89,500  951753852902    Sales   olivia.scott@smartcity.com.au
Jacob Evans     113     654 789 321     7890 Elizabeth St, Melbourne, VIC       0424 567 890     $72,000 654789321013    IT      jacob.evans@smartcity.com.au
Mia Carter      114     321 987 654     1234 Currie St, Adelaide, SA    0425 678 901    $91,000  321987654124    HR      mia.carter@smartcity.com.au
Samuel Morris   115     987 321 654     4567 Oxford St, Brisbane, QLD   0426 789 012    $84,700  987321654235    Finance samuel.morris@smartcity.com.au
Chloe Baker     116     654 159 753     7890 Hindley St, Adelaide, SA   0427 890 123    $79,800  654159753346    Marketing       chloe.baker@smartcity.com.au
Luke Adams      117     789 321 456     1234 George St, Sydney, NSW     0428 901 234    $86,200  789321456457    Operations      luke.adams@smartcity.com.au
Grace Clark     118     753 951 852     4567 North Terrace, Adelaide, SA        0429 012 345     $7om.au
James Lee       119     852 654 753     7890 Queen St8,500      753951852568    IT      grace.clark@smartcity.c, Brisbane, QLD   0430 123 456    $90,300 852654753679    Sales   james.lee@smartcity.com.au
Isabella Young  120     951 852 753     1234 Little Collins St, Melbourne, VIC  0431 234 567     $87,000 951852753790    Finance isabella.young@smartcity.com.au
flag{digging_for_dns_data}
---



## Notes
- This could have been made more difficult with legitimate DNS traffic in the packets. In that case perhaps Solution 2 would have been more robust.
- `jq` syntax (like `sed` and `regex`) is complex and difficult to remember. Large **Language** Models are made for translation, so they're well suited to creating these types of commands. 
- In future I think I'll start with Solution 2 and fall back on bespoke Python code like Solution 1 if that fails. 

---

