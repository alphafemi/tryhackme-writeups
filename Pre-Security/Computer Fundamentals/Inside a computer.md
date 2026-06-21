Nearly every computer system that you can think of includes, in one way or another, the same building blocks. Each part has its own job, and together they make the computer work. Let's have a look at each of these building blocks.


<img width="839" height="559" alt="image" src="https://github.com/user-attachments/assets/74f183fb-cccc-49ba-a8c7-f316f2318a7c" />


## Computer Hardware Fundamentals & Boot Sequence

---

## 1. Core Hardware Components

### The Body Analogy

| Component | Body Part Comparison |
|---|---|
| **Motherboard** | Skeleton & Nervous System |
| **CPU** | Brain (processing) |
| **RAM** | Short-term/Working Memory |
| **Storage (SSD/HDD)** | Long-term Memory |
| **Network Adapter** | Vocal Cords (communication) |
| **PSU** | Heart (pumps power/blood) |
| **Graphics Card** | Visual Cortex |
| **Input/Output** | Senses (eyes, ears, hands) |

---

### 🖥️ Motherboard
- The **central hub** that holds and connects every component
- Contains: **CPU socket**, **RAM slots**, **expansion slots**, various ports
- Everything else plugs into it

 <img width="839" height="559" alt="image" src="https://github.com/user-attachments/assets/5738daea-2a31-4be3-8279-124f69060912" />

---

### 🧠 CPU (Central Processing Unit)
- The **"brain"** — executes instructions
- Modern CPUs have **multiple cores** for parallel processing
- Connects via the **CPU socket**
<img width="839" height="559" alt="image" src="https://github.com/user-attachments/assets/56daeced-cefc-4bfa-9f40-c8503f8b1ff8" />

---

### ⚡ RAM (Random Access Memory)
- **Temporary, fast-access memory** the CPU uses while working
- **Volatile** — all data is lost when power is cut
- Modern types: **DDR5, DDR6**
<img width="839" height="559" alt="image" src="https://github.com/user-attachments/assets/bdedb681-3b67-4ad4-a16e-80bf80c895f7" />

> Like keeping a phone number in your head temporarily — once you stop thinking about it (or fall asleep), it's gone.

---

### 💾 Storage (SSD/HDD)

| | HDD | SSD |
|---|---|---|
| **Technology** | Moving parts | Memory chips, no moving parts |
| **Speed** | Slower | Much faster |
| **Cost** | Cheaper for large capacity | More expensive per GB |
| **Connection** | SATA cables | SATA or PCI Express |
<img width="937" height="559" alt="image" src="https://github.com/user-attachments/assets/b404a5be-a41e-4cb0-9665-d6724eb1d2b4" />

> Permanent storage — like **long-term memory**, data remains even without power.

---

### 📡 Network Adapter
- Allows the computer to **communicate** with other systems
- Comes in **wired** and **wireless** variants
- Often built into the motherboard, or added as an **expansion card**
- Connects via **PCI Express**

---

### 🔌 Power Supply Unit (PSU)
- Supplies **power** to all components
- Converts power from the wall outlet into usable connectors:
  - **Main motherboard connector**
  - **Molex connectors**
<img width="839" height="559" alt="image" src="https://github.com/user-attachments/assets/4554d773-28af-4b79-9040-93994b3bea98" />

> ⚠️ If components demand more power than the PSU can supply, the **system will fail**.

---

### 🎮 Graphics Card (GPU)
- Processes visual data from the OS/programs and **outputs to a monitor**
- Connects via **PCI Express slots**
<img width="897" height="559" alt="image" src="https://github.com/user-attachments/assets/501c44fe-ec8f-452e-8910-13750bcf785b" />

---

### ⌨️ Input/Output Devices

| Type | Examples |
|---|---|
| **Input** | Keyboard, mouse, microphone, scanner |
| **Output** | Monitor, printer, speakers |

**Common connectors:** USB, HDMI, DisplayPort

---
---

## 2. The Boot Sequence

```
[Power Button] 
      │
      ▼
[Step 1: PSU Powers On]
      │
      ▼
[Step 2: Firmware Starts (UEFI/BIOS)]
      │
      ▼
[Step 3: POST — Power-On Self Test]
      │
      ▼
[Step 4: Select Boot Device]
      │
      ▼
[Step 5: Initiate Bootloader]
      │
      ▼
[Operating System Loads into RAM]
```

---

### Step-by-Step Breakdown

#### Step 1 — Press the Power Button
Signal sent to the **PSU**, allowing power to flow to all components.

#### Step 2 — Firmware Starts
The **UEFI** (Unified Extensible Firmware Interface) initialises and manages component startup.

> 📝 **Note:** **BIOS** is the older equivalent of UEFI — UEFI has largely replaced it but you'll still hear the term used.

#### Step 3 — Power-On Self Test (POST)
UEFI runs a test to confirm:
- All required components are **present**
- Everything is **configured correctly**
- Everything is **functioning properly**

#### Step 4 — Select Boot Device
UEFI checks its **ordered list of boot devices** to determine where to look for the Operating System.

#### Step 5 — Initiate Bootloader
- The **bootloader** is launched from the selected boot device
- It transfers the **Operating System into RAM**
- UEFI then **hands over control** to the OS

---

### UEFI/BIOS Summary

| Term | Role |
|---|---|
| **UEFI** | Modern firmware that initialises hardware and starts the boot process |
| **BIOS** | Older firmware doing the same job — now mostly replaced by UEFI |
| **POST** | Self-test ensuring all hardware works correctly |
| **Bootloader** | Loads the OS from storage into RAM |

---
---


```

---
