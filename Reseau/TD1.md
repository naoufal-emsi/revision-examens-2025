Alright. Let’s get into **real explanations** — step by step, with zero assumptions that you already know anything.

---

# 🧠 TOPIC: TCP/IP – IP Protocol (TD-1)

You’re in 3rd year IIR, and this TD is about **IP protocol** — especially **IP fragmentation**, **addressing**, and **subnetting**. This is network engineering stuff. Let’s kill it.

---

## ✅ BASIC CONCEPTS YOU MUST KNOW BEFORE TOUCHING THIS TD

### 🧱 1. What is IP?

**IP (Internet Protocol)** is the system used to deliver **data packets** from one computer to another over a network.

Each data packet is called a **datagram**. Think of it as a **digital envelope** with:

- **Sender address**
    
- **Receiver address**
    
- **Payload (data inside)**
    
- Other control info
    

---

### 📦 2. What is Fragmentation?

Imagine a datagram is **too big** to be sent on the next part of the network. It gets **cut into smaller parts**, called **fragments**. These fragments are then sent one by one and reassembled at the destination.

The **size limit** of the network is called **MTU (Maximum Transmission Unit)**. If a packet is larger than the MTU, it must be fragmented.

---

### 📐 3. Important Terms for Fragmentation

|Field|Meaning|
|---|---|
|**Total Length**|Size of the whole IP datagram (header + data)|
|**Flags**|3 bits. Bit 1 = DF (Don't Fragment), Bit 2 = MF (More Fragments)|
|**Offset**|Position of this fragment's data in the original datagram (in blocks of 8 bytes)|

---

## ⚙️ EXERCISE 1: Fragmenting a 5000-byte datagram with MTU 1500

### 🔢 Given:

- Data size: **5000 bytes**
    
- No options → header size = 20 bytes
    
- Total datagram size = 5000 (data) + 20 (header) = **5020 bytes**
    
- MTU = **1500 bytes** → this is the **maximum** we can send at once
    

**Max data per fragment** = 1500 - 20 = **1480 bytes**

Now fragment the 5000 bytes of data into **pieces of 1480 bytes** or less.

---

### ✅ Step-by-Step Fragmentation:

Let’s divide 5000 bytes of data:

| Fragment | Data Size | Total Length | MF (More Fragments) | Offset         |
| -------- | --------- | ------------ | ------------------- | -------------- |
| Frag 1   | 1480      | 1500         | 1 (yes)             | 0              |
| Frag 2   | 1480      | 1500         | 1                   | 185 (1480 / 8) |
| Frag 3   | 1480      | 1500         | 1                   | 370            |
| Frag 4   | **560**   | **580**      | 0 (last one)        | 555            |

⚠️ Offsets must be multiples of **8** → this is why we divide the byte position by 8.

---

### 🔁 Fragmenting again (MTU = 1000)

Now, a **router** receives fragment 1 and fragment 3 (each has 1480 bytes of data), but the next network’s MTU is 1000.

So now:

- Header: 20 bytes
    
- Max data = 1000 - 20 = **980 bytes**
    

Fragment fragment 1 (which had 1480 bytes of data):

|New Frag|Data Size|Total Length|MF|Offset|
|---|---|---|---|---|
|A1|980|1000|1|0|
|A2|500|520|1|122 (980 / 8)|

Same logic applies to fragment 3.

---

## 🧪 EXERCISE 2: Another Fragmentation Problem

- Datagram: **5600 bytes** of data
    
- Header: 20 bytes → Total: **5620 bytes**
    
- MTU = 1500 → Max data per fragment = 1480 bytes
    

### Fragmenting:

5600 data bytes → divide by 1480:

Fragments:

- 1480 (offset 0)
    
- 1480 (offset 185)
    
- 1480 (offset 370)
    
- **1160** (offset 555)
    

Total length of last = 1160 + 20 = **1180**

---

### Now MTU becomes **820**

Max data per fragment = 820 - 20 = **800 bytes**

Refragment the original fragments into 800-byte fragments.

For example, the 1160-byte fragment becomes:

- 800 bytes → offset: same as before
    
- 360 bytes → offset updated accordingly
    

You get the idea. The router takes incoming fragments and re-fragments them if the MTU is smaller.

---

## 📊 EXERCISE 3: IP Address Analysis

You’re given IPs, you must find:

- Class (A, B, C, D, E)
    
- Default Mask
    
- Network Address
    
- Private/Public?
    
- Number of networks, hosts, etc.
    

Let’s do 2 as examples:

---

### 🔹 IP: 145.32.59.24

- **Class B** (128–191 → first byte = 145)
    
- **Mask**: 255.255.0.0
    
- **Network**: 145.32.0.0
    
- **Private?** No → **public**
    
- **# Machines**: 2¹⁶ - 2 = **65534**
    

---

### 🔹 IP: 10.10.12.12

- **Class A** (1–126)
    
- **Mask**: 255.0.0.0
    
- **Network**: 10.0.0.0
    
- **Private** → YES (10.0.0.0/8 is private)
    

---

## 🧠 EXERCISE 4: Subnetting 192.168.90.0

Want **4 subnets**, each with **max 32 hosts**.

What to do:

- Hosts per subnet: needs 5 bits (2⁵ = 32)
    
- So subnet mask must leave 5 bits for hosts → 32 - 5 = **/27**
    
- Subnets:
    

|Subnet|Network Address|Range|Broadcast|
|---|---|---|---|
|1|192.168.90.0/27|.1 → .30|192.168.90.31|
|2|192.168.90.32/27|.33 → .62|192.168.90.63|
|3|192.168.90.64/27|.65 → .94|192.168.90.95|
|4|192.168.90.96/27|.97 → .126|192.168.90.127|

---

## 📌 EXERCISE 5: Plan Addressing for 149.168.0.0/16

- /16 = 65534 hosts → huge range
    
- You decide how to divide it: /24 subnets = 256 subnets with 254 hosts each
    

You can give each department or building a /24 block:

- 149.168.1.0/24
    
- 149.168.2.0/24
    
- ...
    

Or be more efficient by subnetting based on need (like in next exercise).

---

## 🧩 EXERCISE 6: Emsi Site Addressing with 192.168.16.0/21

/21 = 2048 IPs → enough for all sites.

Allocate per site:

|Site|Hosts|Required bits|Subnet Mask|Subnet|
|---|---|---|---|---|
|Casa|60|6 bits (64)|/26|192.168.16.0/26|
|Rabat|40|6 bits (64)|/26|192.168.16.64/26|
|Marrakech|25|5 bits (32)|/27|192.168.16.128/27|
|Tanger|25|5 bits (32)|/27|192.168.16.160/27|

---

## ✅Alright, Master — now let’s **slow down and break this into simpler parts**, starting from **zero** knowledge.

We’re redoing:

- ✅ **Exercice 3**: IP classes, masks, types, number of hosts
    
- ✅ **Exercice 4**: Subnetting for 4 subnets with max 32 hosts
    
- ✅ **Exercice 5**: Plan an addressing scheme for a big network
    

No BS. 100% clear. Let's go.

---

# ✅ EXERCICE 3 — IP Address Analysis

You're given some IPs, and for each one you need to find:

|Field|What it means|
|---|---|
|**Class**|A, B, C, D, or E — based on first number in IP|
|**Mask**|The default subnet mask for the class (unless specified otherwise)|
|**Network Address**|The first IP in the network (host part becomes 0)|
|**# of Networks**|Only makes sense if you're subnetting — we’ll treat it as N/A here|
|**# of Hosts**|How many usable machines (2^host_bits - 2)|
|**Private/Public**|Is it a public IP or a private one?|

---

### 🎯 Step-by-Step Formula

1. **Class detection**:
    
    - 1 to 126 → Class A
        
    - 128 to 191 → Class B
        
    - 192 to 223 → Class C
        
    - 224 to 239 → Class D (multicast)
        
    - 240+ → Class E (experimental)
        
2. **Default Masks**:
    
    - Class A → /8 → 255.0.0.0
        
    - Class B → /16 → 255.255.0.0
        
    - Class C → /24 → 255.255.255.0
        
3. **Network address**: Set all host bits to 0 (e.g. 192.168.1.34 → 192.168.1.0 if /24)
    
4. **Host count**:
    
    hosts=2host bits−2\text{hosts} = 2^{\text{host bits}} - 2
    - We subtract 2 because:
        
        - One is reserved for **network address**
            
        - One is reserved for **broadcast address**
            

---

### 🔍 Now let’s do them all:

#### 1. **145.32.59.24**

- First octet = 145 → Class **B**
    
- Default Mask: **255.255.0.0** → /16
    
- Network Address: **145.32.0.0**
    
- Host bits = 32 - 16 = 16
    
- Hosts = 2¹⁶ - 2 = **65534**
    
- Is it private? No. → **Public**
    

---

#### 2. **200.42.129.16**

- 200 → Class **C**
    
- Mask: **255.255.255.0** → /24
    
- Network: **200.42.129.0**
    
- Host bits = 8 → 2⁸ - 2 = **254**
    
- Public? **Yes**
    

---

#### 3. **10.10.12.12**

- 10 → Class **A**
    
- Mask: **255.0.0.0** → /8
    
- Network: **10.0.0.0**
    
- Host bits = 32 - 8 = 24
    
- Hosts = 2²⁴ - 2 = **16,777,214**
    
- Private? **Yes** (10.0.0.0/8 is private)
    

---

#### 4. **172.22.177.13**

- 172 → Class **B**
    
- Mask: 255.255.0.0 → /16
    
- Network: 172.22.0.0
    
- Host bits: 16 → 2¹⁶ - 2 = **65534**
    
- Private? YES → 172.16.0.0 to 172.31.255.255 = private
    

---

#### 5. **123.12.45.77**

- 123 → Class A
    
- Mask: 255.0.0.0 → /8
    
- Network: 123.0.0.0
    
- Host bits = 24 → 2²⁴ - 2 = **16,777,214**
    
- Private? **NO**
    

---

#### 6. **1.102.45.177**

- 1 → Class A
    
- Mask: 255.0.0.0
    
- Network: 1.0.0.0
    
- Host bits: 24 → 2²⁴ - 2 = 16,777,214
    
- Private? NO
    

---

#### 7. **192.134.16.123**

- 192 → Class C
    
- Mask: 255.255.255.0 → /24
    
- Network: 192.134.16.0
    
- Host bits = 8 → 2⁸ - 2 = 254
    
- Private? NO (Private 192.168.x.x only)
    

---

### 🧾 FINAL TABLE:

|Address|Class|Mask|Network Address|Hosts|Private?|
|---|---|---|---|---|---|
|145.32.59.24|B|255.255.0.0|145.32.0.0|65534|No|
|200.42.129.16|C|255.255.255.0|200.42.129.0|254|No|
|10.10.12.12|A|255.0.0.0|10.0.0.0|16777214|**Yes**|
|172.22.177.13|B|255.255.0.0|172.22.0.0|65534|**Yes**|
|123.12.45.77|A|255.0.0.0|123.0.0.0|16777214|No|
|1.102.45.177|A|255.0.0.0|1.0.0.0|16777214|No|
|192.134.16.123|C|255.255.255.0|192.134.16.0|254|No|

---

# ✅ EXERCICE 4 — Subnetting 192.168.90.0

## 🎯 Goal:

- Split **192.168.90.0** into **4 subnets**
    
- Each subnet supports **up to 32 machines (hosts)**
    

---

## 🧠 Steps:

### 1. How many bits do you need to represent 32 hosts?

25=32→need∗∗5bits∗∗forhostpart2^5 = 32 → need **5 bits** for host part

That leaves:

32−5=∗∗27bits∗∗fornetwork32 - 5 = **27 bits** for network

So we need a **/27 subnet mask**

- Subnet Mask: **255.255.255.224**
    

Because:

/27→8+8+8+3→lastbyte:11100000=224/27 → 8 + 8 + 8 + 3 → last byte: 11100000 = 224

---

### 2. What’s the increment per subnet?

Formula:

Increment=2host bits=25=32\text{Increment} = 2^{\text{host bits}} = 2^5 = 32

---

### 3. Build the subnets:

|Subnet #|Network Address|Usable Hosts|Broadcast Address|
|---|---|---|---|
|1|192.168.90.0/27|.1 to .30|192.168.90.31|
|2|192.168.90.32/27|.33 to .62|192.168.90.63|
|3|192.168.90.64/27|.65 to .94|192.168.90.95|
|4|192.168.90.96/27|.97 to .126|192.168.90.127|

Each has exactly **32 IPs**:

- 1 for **network**
    
- 1 for **broadcast**
    
- **30 usable** hosts
    

---

# ✅ EXERCICE 5 — Address Plan for 149.168.0.0/16

## 🎯 Goal:

Propose a plan for using the network **149.168.0.0/16**

- It’s a **Class B** network
    
- Default mask: /16 → 255.255.0.0
    
- Hosts = 2¹⁶ - 2 = **65534** total IPs
    

---

## Two ways to go:

---

### 🔹 Option 1: Subdivide into /24 networks (simple)

Each /24 subnet:

- Has 256 IPs → 254 usable
    
- Good for small networks like departments or offices
    

You can create:

28=256 subnets2^{8} = 256 \text{ subnets}

Example plan:

|Subnet #|Subnet Address|Hosts|
|---|---|---|
|1|149.168.1.0/24|254|
|2|149.168.2.0/24|254|
|3|149.168.3.0/24|254|
|...|...|...|
|256|149.168.255.0/24|254|

---

### 🔹 Option 2: Variable-length subnetting (better)

Give **bigger subnets** to big teams, smaller to small teams.

Example:

- HR → 50 hosts → /26
    
- IT → 200 hosts → /24
    
- Admin → 10 hosts → /28
    

You assign **just enough space** for each.

---

Let me know if you want a real address plan example for a company (with teams, buildings, etc.) — I can build that too.

---

