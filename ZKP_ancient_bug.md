# Kerberoasting 

## The Shocking Reality

This is a vulnerability that **shouldn't exist in 2025**. Yet it's been actively exploited for over a decade, including in the May 2024 ransomware attack on Ascension Health hospital system. Microsoft calls it "low tech, high-impact."

**The problem?** Ancient cryptography from the 1990s is still running in modern Windows networks.

---

## What's Active Directory and Why Should You Care?

**Active Directory (AD)** is Microsoft's system that controls who can access what on Windows networks. Think of it as the gatekeeper for almost every Windows corporate network.

### How It Works (Simplified)

1. You (an employee) want to access a network resource (file server, printer, etc.)
2. You ask the Active Directory server: "Can I access this?"
3. AD gives you an encrypted "ticket" to present to the resource
4. The resource checks your ticket and lets you in

**The critical point:** The ticket is encrypted using the Service's password.

### The Security Model

AD is supposed to protect companies from disasters. If a hacker tricks one employee into clicking a bad link, they get control of that ONE laptop. But they shouldn't be able to take over critical servers. AD is the wall that stops them.

**Or at least it should be.**

---

## What Is Kerberoasting?

It's an attack where hackers steal encrypted tickets and crack the passwords offline. Here's how it works:

### The Attack (Step by Step)

**Step 1: Get Inside**

- Hacker compromises one employee laptop (maybe through a phishing email)

**Step 2: Request Tickets**

- From that laptop, ask AD for tickets to various Services
- AD happily hands over encrypted tickets (this is normal behavior!)
- Anyone on the network can request these tickets

**Step 3: Take Tickets Home**

- Export the encrypted tickets to the hacker's own computers
- Now they can crack passwords offline at their leisure
- No one on the corporate network even knows this is happening

**Step 4: Crack the Passwords**

- Try millions/billions of password guesses
- When you find the right password, you can control that Service
- Move laterally through the network
- Deploy ransomware

---

## Why Does This Work? The Design Flaw

The fundamental problem: **Any random user can request an encrypted ticket for any Service, even if they don't have access.**

Think about it:

- The ticket is encrypted with the Service's password
- The attacker doesn't know the password yet
- But they can try to crack it offline because they have the encrypted ticket
- It's like giving a burglar a copy of your locked safe to take home and work on

**This invites offline cracking attacks.**

---

## The Two Layers of Disaster

### Layer 1: Weak Passwords (Human Error)

**The good scenario:** Services should use randomly-generated cryptographic keys (long, random, impossible to guess). Microsoft even has systems to auto-generate and rotate these.

**The bad reality:** Network administrators sometimes set up Services using regular user accounts with human-generated passwords like "Summer2024!" or "CompanyName123".

**Why this is terrible:** Human passwords are weak and guessable.

### Layer 2: Ancient Cryptography (Microsoft's Fault)

Even with weak passwords, strong encryption could make cracking difficult. But here's where it gets ridiculous...

#### The "Modern" Protection (Not Great)

When using AES encryption mode:

- Uses AES encryption
- Uses PBKDF2 with 4,096 iterations for key derivation
- Makes cracking slower (but not slow enough)

**Performance:** RTX 5090 GPU can try **6.8 million passwords per second**

Not great, but at least there's some defense.

#### The Ancient Disaster (RC4 Mode)

Here's the killer: **AD still supports RC4 encryption from the 1990s as a fallback.**

When a Service isn't configured for modern encryption:

- Falls back to RC4 encryption
- Uses NT hash (basically one iteration of MD4)
- No salt (attackers can pre-compute password tables)

**Performance:** Same RTX 5090 can try **4.18 BILLION passwords per second**

**That's 1,000× faster than AES mode.**

---

## A Real-World Comparison

Let's say a Service password is "CompanyName2024!"

### With AES (Modern Mode):

```
Password cracking speed: 6.8 million/second
Time to crack: Maybe hours to days
Defense: Moderately difficult
```

### With RC4 (Legacy Mode):

```
Password cracking speed: 4.18 BILLION/second  
Time to crack: Seconds to minutes
Defense: Basically none
```

**The catastrophe:** A password that might take hours to crack with AES can be cracked in seconds with RC4.

---

## Timeline of Failure

- **1989:** Kerberos protocol invented
- **1999:** Microsoft Active Directory debuts (using old crypto)
- **2014:** Tim Medin discovers and names "Kerberoasting"
- **2014-2024:** Attack becomes well-known, documented everywhere
- **2024:** Still being used in ransomware attacks (Ascension Health)
- **2025:** RC4 mode still supported by default

**This vulnerability is over a decade old with a catchy name, and it's still actively exploited.**

---

## Why Hasn't Microsoft Fixed This?

### What Microsoft Recommends (Weakly):

1. Use automated key assignment for Services (good, but optional)
2. If you can't do that, use "really good long passwords" (relying on humans)
3. If you can't do that, please disable RC4 (still optional!)

### What Microsoft Should Do (But Doesn't):

1. **Ban RC4 entirely** - it's 2025, not 1999
2. **Force Services to use strong random keys**
3. **Make the system obnoxious until admins upgrade**
4. **Block weak human passwords for Service accounts**

**The problem:** Microsoft makes it all optional. System administrators don't upgrade. Hackers exploit the gaps.

---

### 1. No Detection

- Requesting tickets is normal behavior
- Cracking happens offline (outside the network)
- Defenders can't see the attack happening

### 2. Any User Can Do It

- You don't need special privileges
- Compromise ONE laptop → request ALL Service tickets
- The weakest link breaks everything

### 3. Time Is Not Your Friend

- Attacker can work offline for weeks/months
- GPU farms make cracking absurdly fast
- Even "moderate" passwords fall quickly

### 4. Legacy Crypto Kills You

- If even ONE Service uses RC4, it's vulnerable
- Admins might not know which mode is active
- RC4 makes cracking 1,000× faster

---

## The Real Victim: Ascension Health

In May 2024, ransomware attackers used Kerberoasting against Ascension Health hospital system:

1. Compromised initial access (likely phishing)
2. Used Kerberoasting to crack Service account passwords
3. Gained control of critical systems
4. Deployed ransomware
5. Caused massive disruption to healthcare services

**This shouldn't happen in 2025.**

---

## The Core Lessons

1. **Old crypto is dangerous:** RC4 from the 1990s should be dead but isn't
2. **Offline attacks are powerful:** Once attackers steal encrypted data, they can crack it at leisure
3. **Defense in depth matters:** One weak Service account can compromise everything
4. **Legacy support is a security nightmare:** Supporting old systems for compatibility creates vulnerabilities

---


- Admins don't always follow best practices
- One weak password + RC4 = instant compromise

It's "low tech" (just password cracking) but "high impact" (full network takeover and ransomware).
