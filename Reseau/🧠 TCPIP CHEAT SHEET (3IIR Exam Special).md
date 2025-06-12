---

---
 ---
 
## 📦 IP ADDRESS CLASSES

|Class|1st Octet Range|Default Mask|Private Range|# Hosts|
|---|---|---|---|---|
|A|1 – 126|255.0.0.0 (/8)|10.0.0.0 → 10.255.255.255|~16M|
|B|128 – 191|255.255.0.0 (/16)|172.16.0.0 → 172.31.255.255|~65K|
|C|192 – 223|255.255.255.0 (/24)|192.168.0.0 → 192.168.255.255|254|
|D|224 – 239|Reserved (Multicast)|N/A|N/A|
|E|240 – 255|Reserved (Experimental)|N/A|N/A|

🧠 **Tip**: 127.x.x.x is for loopback (localhost)

---

## 📏 CIDR → MASK CONVERSIONS

|CIDR|Subnet Mask|# IPs|# Usable Hosts|
|---|---|---|---|
|/8|255.0.0.0|16,777,216|16,777,214|
|/16|255.255.0.0|65,536|65,534|
|/24|255.255.255.0|256|254|
|/25|255.255.255.128|128|126|
|/26|255.255.255.192|64|62|
|/27|255.255.255.224|32|30|
|/28|255.255.255.240|16|14|
|/29|255.255.255.248|8|6|
|/30|255.255.255.252|4|2|

🧠 **Shortcut**: `256 - subnet mask last byte = block size`

---

## 🧮 SUBNETTING: HOW TO DO IT FAST

1. **Want N hosts?**
    
    - Use formula: `2^n - 2 ≥ N` → find `n`
        
    - Hosts = 2^host_bits - 2
        
2. **Want N subnets?**
    
    - Use formula: `2^n ≥ N` → find `n`
        
    - Add `n` to the default CIDR
        
3. **Range of IPs:**
    
    - Start = network address + 1
        
    - End = broadcast address - 1
        
    - Broadcast = last IP of block
        

---

## 📬 IP FRAGMENTATION

- **MTU**: Max size in bytes for transmission
    
- **Header**: 20 bytes (if no options)
    
- **Offset**: position of data chunk (in 8-byte units)
    
- **Flags**:
    
    - DF = Don’t Fragment
        
    - MF = More Fragments
        

🧠 **Fragment Formula**:

```
Max Data Per Fragment = MTU - Header
Offset = (previous offset + data size) / 8
```

---

## 🛣️ ROUTING TABLE BASICS

|Field|Meaning|
|---|---|
|Network Addr|Destination subnet|
|Mask|Netmask used to match|
|Next Hop|Where to send the packet|
|Interface|Interface used for forwarding|

🧠 **Router decision process:**

1. Longest match wins (most specific subnet mask)
    
2. If no match → check default route (`0.0.0.0/0`)
    
3. If still nothing → packet dropped
    

---

## 🚨 COMMON MISTAKES (DON’T DO THIS)

|Mistake|Fix|
|---|---|
|Using .0 or .255 as host IP|These are **network** and **broadcast**|
|Mixing up /28 and /27|Memorize: /27 = 32 IPs, /28 = 16 IPs|
|Forgetting to subtract 2 from host count|Always subtract for network & broadcast|
|Default route points to loopback|Must point to a **real** next hop IP|
|Using wrong interface for destination|Verify IP is in subnet of that interface|

---

## 💡 MNEMONICS & TRICKS

- **“CIDR 3 Steps Down = Half IPs”** → /24 (256) → /25 (128) → /26 (64)...
    
- **“Class A Starts at 1, Not 0”**
    
- **“Private IP = 10, 172.16-31, 192.168”** → memorize like phone code
    

---

## 🎯 EXAM STRATEGY

|Situation|Action|
|---|---|
|IP address with mask|Identify subnet & range|
|Routing table + source/dest IP|Find subnet match → interface|
|Need 4 subnets of 32 hosts|Use /27 (30 usable hosts)|
|Fragmentation needed|Use MTU - 20 → chunk, calc offset|
|Ping between LANs/Servers|Trace path hop-by-hop|

🧠 **Recognize key words**:

- “mask” → subnetting
    
- “next hop” → routing table
    
- “fragment” or “MTU” → fragmentation
    
- “routeur reçoit un paquet…” → routing logic
    

---

📎 Want a PDF version? Want it as a printable flashcard sheet? I can export this for you or turn it into a visual memory map.

Say the word.