## Cloud Computing Fundamentals

---

## 1. What is Cloud Computing?

### The Core Idea
Instead of running an application on **one computer in one location**, cloud computing lets you use **computing resources over the internet** — making applications more **accessible, reliable, and scalable**.

```
[Single Laptop/Server]  ──evolves into──►  [Cloud — Resources Anywhere]
   Limited capacity                          Globally accessible
   Single point of failure                   Highly available
```

---
---

## 2. Evolution: Physical Servers → Cloud

```
Physical Servers ──► Virtualization ──► Hosted/Managed Servers ──► Cloud Computing
   (one app per box)   (VMs, shared       (outsourced hardware)      (on-demand,
                         hardware)                                    global, scalable)
```

> At each stage, the goal was the same: **reduce costs**, **improve efficiency**, and make scaling **easier**.

---
---

## 3. Cloud Benefits & Characteristics

| Benefit | Description |
|---|---|
| 📈 **Scalability** | Scale resources up or down as needs change |
| ⚡ **On-Demand Self-Service** | Create/remove resources instantly — no hardware wait |
| 💰 **Pay-As-You-Go** | Charged based on usage, not upfront costs |
| 🔒 **Security** | Cloud providers maintain strong infrastructure security |
| 🟢 **High Availability** | Application keeps running even if part of the system fails |
| 🌍 **Global Access** | Accessible to users anywhere in the world |

---
---

## 4. Cloud Deployment Types

```
┌──────────────────────────────────────────────────┐
│                  CLOUD DEPLOYMENT                  │
├──────────────┬──────────────────┬─────────────────┤
│ Public Cloud │  Private Cloud    │  Hybrid Cloud    │
└──────────────┴──────────────────┴─────────────────┘
```

| Type | Description | Used By |
|---|---|---|
| **Public Cloud** | Shared infrastructure, affordable, easy to scale | Startups, websites, global apps |
| **Private Cloud** | Dedicated infrastructure, more control & compliance | Banks, healthcare, government |
| **Hybrid Cloud** | Mix of private (sensitive data) + public (scaling) | E-commerce during high-demand periods |

---
---

## 5. Cloud Service Models

### The Apartment Rental Analogy

```
IaaS                    PaaS                     SaaS
─────                    ─────                     ─────
Rent empty land       Rent a furnished           Rent a fully
Build your own house    apartment                  serviced hotel room
(You manage most)      (Provider manages           (Provider manages
                         structure, you             EVERYTHING)
                         manage interior)
```

| Model | What You Manage | What Provider Manages | Example |
|---|---|---|---|
| **IaaS** (Infrastructure as a Service) | OS, applications | Physical hardware, networking, storage | AWS EC2, Azure VMs |
| **PaaS** (Platform as a Service) | Just your application/code | Infrastructure + OS | Heroku, AWS Elastic Beanstalk |
| **SaaS** (Software as a Service) | Nothing — just use it | Everything | Gmail, Zoom, Dropbox |

### Responsibility Comparison

```
                  IaaS        PaaS        SaaS
Application      [YOU]       [YOU]       [Provider]
Runtime          [YOU]       [Provider]  [Provider]
OS                [YOU]       [Provider]  [Provider]
Virtualization    [Provider]  [Provider]  [Provider]
Hardware          [Provider]  [Provider]  [Provider]
```

---
---

## 6. Major Cloud Vendors

| Vendor | Known For |
|---|---|
| **AWS** (Amazon Web Services) | Industry leader — largest range of services, global reach |
| **Microsoft Azure** | Strong in enterprise & hybrid cloud |
| **Google Cloud Platform (GCP)** | Data analytics, AI, machine learning |
| **Alibaba Cloud** | Major player in Asia |
| **IBM Cloud** | Hybrid cloud, AI-driven solutions |
| **Oracle Cloud** | Enterprise applications & databases |

> **AWS remains the most popular** due to its vast infrastructure and broad support for businesses of all sizes.

---
---

## 7. Real-World Cloud Use Cases

| Company | How They Use the Cloud |
|---|---|
| **Netflix** | Runs entirely on AWS — scales globally, stays online during peak demand |
| **Spotify** | Handles millions of songs/users, scales quickly for new releases |
| **Instagram** | Stores massive photo/video data, delivers fast worldwide |
| **Online Stores** | Handle traffic spikes (e.g. Black Friday) without permanent infrastructure |

### Why Companies Choose the Cloud
```
✅ Scale easily
✅ Reduce costs
✅ Stay reliable
✅ Focus on the product, not hardware management
```

---
---
Key Terminology
Let's quickly review some concepts we learned in this room:

Public Cloud 
Cloud services you access over the internet that many people and companies share.
Private Cloud 
A cloud built just for one company, so they have more control and security.
Hybrid Cloud
A mix of public and private clouds that can work together and share data.
IaaS
A service where you rent basic computer parts like servers and storage from the cloud.
PaaS
A service that gives you a ready-to-use environment to build and run apps without managing servers.
SaaS
Software you use online without installing anything, like Gmail or Zoom.
EC2
Amazon’s cloud computers that you can quickly create, use, and resize whenever you need them.
We also concluded that the key benefits of cloud computing are:

Scalability
On-demand self-service
Pay only for what you use
Security
High availability
Global access
