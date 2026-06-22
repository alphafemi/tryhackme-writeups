## Operating Systems Fundamentals

---

## 1. What is an Operating System?

### Definition
The **OS** is the core software that **coordinates everything** on a computer — sitting between the user, applications, and hardware.

```
┌─────────────────┐
│      USER         │
├─────────────────┤
│   APPLICATIONS     │
├─────────────────┤
│ OPERATING SYSTEM   │  ← The coordinator
├─────────────────┤
│  PHYSICAL HARDWARE │
└─────────────────┘
```

---

### The Airport Analogy ✈️

| Computer Component | Airport Equivalent |
|---|---|
| **Hardware** (CPU, RAM, storage) | Runways, airplanes, fuel systems, radar |
| **Applications** (browser, games) | Airlines and passengers |
| **Operating System** | Air Traffic Control |

> The OS **schedules**, **manages traffic**, **resolves conflicts**, and **ensures safety** — just like ATC does for an airport.

### Why Do We Need an OS?
Without one, **every application** would need direct control of the CPU, memory, files, and devices — causing constant conflicts. The OS acts as the **central organizer**.

---
---

## 2. System Privilege Layers

```
┌──────────────────────────────────┐
│           USER SPACE              │  ← Applications (browser, games)
│   (Restricted — no direct         │
│    hardware access)               │
├──────────────────────────────────┤
│   System Calls (requests) ↕        │
├──────────────────────────────────┤
│          KERNEL SPACE              │  ← The OS Core
│   (Privileged — full hardware     │
│    access)                        │
└──────────────────────────────────┘
```

| Layer | Access Level | Who/What Runs Here |
|---|---|---|
| **Kernel Space** | Full, unrestricted hardware access | The kernel itself |
| **User Space** | Restricted — must request via **system calls** | Regular applications |

### Airport Analogy Continued
- **Kernel Space** = The **control tower** — only trusted controllers (kernel) can touch the equipment
- **User Space** = **Airlines/passengers** on the ground — they radio requests (system calls) instead of touching equipment directly

> 🔒 This separation means **one faulty app can't crash the whole system**.

---
---

## 3. Core OS Responsibilities

| Responsibility | What It Does | Example |
|---|---|---|
| **Process Management** | Creates, schedules, prioritises, terminates programs | Running browser + music player + social media without freezing |
| **Memory Management** | Allocates/protects RAM, uses virtual memory when low | Multiple apps run isolated, without interfering |
| **File System Management** | Organises files, directories, permissions, metadata | Creating folders, saving photos, setting "read only" |
| **User Management** | Handles accounts, authentication, permissions | Logging in with a password; files hidden from other users |
| **Device Management** | Loads drivers, provides hardware abstraction | Plugging in a mouse/printer and it "just works" |

```
[Process Mgmt] [Memory Mgmt] [File System] [User Mgmt] [Device Mgmt]
        └──────────────┬──────────────┴─────────────┘
                  OPERATING SYSTEM CORE
```

---
---

## 4. Operating System Security

The OS is the **first line of defence** — before any antivirus or firewall even gets involved.

| Security Function | Description |
|---|---|
| **Authentication** | Verifies identity (passwords, biometrics) |
| **Permissions** | Controls what users/apps can read, write, execute |
| **Isolation** | Keeps each process in its own protected space |
| **System Protection** | Guards critical files/settings from unauthorized changes |

---
---

## 5. OS Interfaces — GUI vs CLI

| | GUI (Graphical User Interface) | CLI (Command-Line Interface) |
|---|---|---|
| **Interaction** | Icons, windows, menus — click-based | Text commands — typed |
| **Ease of Use** | Beginner-friendly | Requires command knowledge |
| **Precision/Speed** | Less precise, more visual | Highly precise & fast for advanced tasks |
| **Analogy** | Tapping a destination on a map app | Typing exact GPS coordinates |

```
Same Task: "Show contents of home directory"

GUI:  [Click] File Explorer → [Click] Home folder → See files
CLI:  $ ls ~                                      → See files
```

---
---

## 6. Types of Operating Systems

| OS Type | Primary Use Case | Key Characteristics |
|---|---|---|
| **Desktop** | Personal computers, gaming, work | Rich GUI, multitasking, user-focused |
| **Server** | Web hosting, databases, cloud | Headless (no GUI), high uptime, multi-user |
| **Mobile** | Smartphones, tablets | Touch UI, power-efficient, app sandboxing |
| **Embedded** | IoT, cars, routers, smart TVs | Tiny footprint, limited hardware |
| **Virtual/Cloud** | VMs, containers, cloud instances | Lightweight, scalable, rapid deployment |

---
---

## 7. Real-World Operating System Families

### 🖥️ Desktop
| OS | Examples |
|---|---|
| **Windows** | Windows 10, Windows 11 |
| **macOS** | Sonoma (14), Sequoia (15), Tahoe (26) |
| **Linux** | Ubuntu, Debian, Fedora |

### 🖧 Server
| OS | Examples |
|---|---|
| **Windows Server** | 2016, 2019, 2022, 2025 |
| **Linux** | Ubuntu Server, Debian, CentOS, Red Hat |
| **Unix** | IBM AIX, Oracle Solaris |

### 📱 Mobile
| OS | Examples |
|---|---|
| **Android** | Android 14–16 + manufacturer versions |
| **iOS** | iOS 17, 18, 26 |

### 🔌 Embedded/IoT
| OS | Examples |
|---|---|
| **Embedded Linux** | OpenWrt, Ubuntu Core, Yocto Project |
| **Real-Time OS (RTOS)** | FreeRTOS, VxWorks, QNX |

### ☁️ Virtual/Cloud
| OS | Examples |
|---|---|
| **Cloud/VM** | Ubuntu LTS, Amazon Linux, Rocky Linux |
| **Container-Optimized** | Alpine Linux, Bottlerocket AWS, Flatcar Linux |

---
---

## 8. Why So Many Operating Systems?

```
[Laptop]   → Needs user-friendly multitasking
[Server]   → Needs stability, security, 24/7 uptime
[Mobile]   → Needs power efficiency, hardware integration
[Embedded] → Needs lightweight, specialized function
```

> No single OS fits every situation — different goals (ease of use, performance, security, openness) led to an **entire ecosystem** of operating systems.

---
---

### Key Terminology
Let’s recap the core terms you’ve learned. These definitions will help solidify your understanding before moving on to further learning.

### Operating system (OS)
The core software that manages hardware, applications, and all system resources.

### Kernel space
The OS’s highly privileged area with direct hardware access, and the home of the kernel, which directly manages hardware and system resources.

### User space
The area where regular applications run with limited permissions for safety and system stability.

### Graphical user interface (GUI)
The visual part of the OS, windows, icons, and menus, that lets you interact through clicking and tapping.

### Command-line interface (CLI)
A text-based interface where you type commands to control the system with precision and speed.
