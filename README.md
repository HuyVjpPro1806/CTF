<div align="center">

# 🧩 CTF Writeups

### Reverse Engineering • Blockchain Investigation • Evidence-Based Solving

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:020617,45:1d4ed8,100:22c55e&height=190&section=header&text=CTF%20Writeups&fontSize=52&fontColor=ffffff&animation=fadeIn&fontAlignY=36&desc=Reverse%20Engineering%20%7C%20Blockchain%20Tracing%20%7C%20Technical%20Writeups&descAlignY=60&descSize=17" />

<br>

![CTF](https://img.shields.io/badge/CTF-Writeups-a855f7?style=for-the-badge\&logo=hackthebox\&logoColor=white)
![Reverse Engineering](https://img.shields.io/badge/Reverse%20Engineering-PE%20Analysis-2563eb?style=for-the-badge\&logo=windows\&logoColor=white)
![Blockchain](https://img.shields.io/badge/Blockchain-USDT%20Tracing-22c55e?style=for-the-badge\&logo=bitcoin\&logoColor=white)
![IDA](https://img.shields.io/badge/Tool-IDA%20Pro-f97316?style=for-the-badge)
![Writeups](https://img.shields.io/badge/Style-Evidence%20Based-0f172a?style=for-the-badge)

</div>

---

## 📌 About This Repository

This repository contains my personal CTF writeups from competitions I have participated in.

The current writeups mainly focus on two areas:

* **Reverse Engineering**: analyzing Windows PE binaries, anti-debug checks, program logic, and flag validation routines.
* **Blockchain Investigation**: tracing USDT transactions, following fund movement across wallets, and identifying the last traceable on-chain hop.

The goal of this repository is not only to store flags, but to document the full solving process:

```text
Understand the challenge
→ Analyze the evidence
→ Identify the logic
→ Reproduce the solution
→ Verify the final answer
```

---

## 🗂️ Repository Structure

```text
CTF/
├── LYKNCTF-2026/
│   └── writeup_funnull_usdt_trace.md
│
├── UTECTF2026/
│   ├── PE01 — Write-up.md
│   └── PE02 - Write-up.md
│
└── README.md
```

---

## 🧭 Writeup Index

| Event        | Challenge                                                          | Category                 | Main Technique                                          | Status |
| ------------ | ------------------------------------------------------------------ | ------------------------ | ------------------------------------------------------- | ------ |
| LYKNCTF 2026 | [FunNull USDT Trace](./LYKNCTF-2026/writeup_funnull_usdt_trace.md) | Blockchain Investigation | TRON USDT tracing, OFAC entity attribution              | ✅ Done |
| UTECTF 2026  | [PE01](./UTECTF2026/PE01%20%E2%80%94%20Write-up.md)                | Reverse Engineering      | PE32 analysis, anti-debug bypass, equation solving      | ✅ Done |
| UTECTF 2026  | [PE02](./UTECTF2026/PE02%20-%20Write-up.md)                        | Reverse Engineering      | PE32 analysis, anti-debug bypass, permutation reversing | ✅ Done |

---

## 🔍 Current Writeups

### 🪙 LYKNCTF 2026 — FunNull USDT Trace

This writeup analyzes a suspicious USDT transaction on the **TRON** network.

The investigation follows the money flow through multiple wallets and identifies:

* The original transaction
* The USDT TRC-20 token movement
* The intermediate laundering hops
* The last traceable on-chain transaction
* The sanctioned entity related to the wallet

Main skills practiced:

```text
Blockchain tracing
Wallet-to-wallet correlation
Transaction timeline analysis
Exchange wallet attribution
OFAC sanctions verification
```

---

### 🧬 UTECTF 2026 — PE01

This writeup focuses on a Windows PE reverse engineering challenge.

The binary asks for two keys. By analyzing the program in IDA, the writeup identifies anti-debug logic using `IsDebuggerPresent()` and then follows the real validation function.

The core idea is that the program checks two equations involving `key1` and `key2`. After dumping the required constants from the binary, the keys can be recovered by solving the equations.

Main skills practiced:

```text
PE32 static analysis
IDA pseudocode analysis
Anti-debug bypass
Global variable inspection
Big integer operation analysis
Equation solving with Python
```

---

### 🧬 UTECTF 2026 — PE02

This writeup is another PE reverse engineering challenge.

The program asks the user to input a flag. After bypassing anti-debug logic, the validation routine shows that the input is copied, transformed, and compared against a scrambled target string.

The solution is to understand the permutation logic and reverse it to recover the original flag.

Main skills practiced:

```text
PE32 analysis
String searching in IDA
Anti-debug bypass
Input transformation analysis
Array and table inspection
Reverse permutation scripting
```

---

## 🧰 Tools Used

| Tool                           | Purpose                                 |
| ------------------------------ | --------------------------------------- |
| **IDA**                        | Static and dynamic reverse engineering  |
| **Kali / Ubuntu**              | Analysis environment                    |
| **file**                       | Identify binary format and architecture |
| **Python**                     | Write solver scripts                    |
| **TRON / blockchain explorer** | Trace USDT TRC-20 transactions          |
| **OFAC Sanctions Search**      | Verify sanctioned entity information    |

---

## 🧪 Solving Methodology

### Reverse Engineering Workflow

```text
Run the program
→ Identify expected input
→ Check binary format
→ Load into IDA
→ Locate main logic
→ Bypass anti-debug checks
→ Analyze validation function
→ Extract constants / arrays
→ Write solver script
→ Verify flag in the program
```

### Blockchain Investigation Workflow

```text
Start from given transaction hash
→ Identify chain and token
→ Follow incoming and outgoing transfers
→ Correlate amount and timestamp
→ Continue tracing wallet hops
→ Stop when attribution becomes unreliable
→ Verify entity using trusted sources
→ Build final flag format
```

---

## 🧠 What I Focus On

```text
Evidence first, flag second.
```

For each challenge, I try to clearly document:

* What the challenge gives
* What tool was used
* What evidence was found
* Why a specific direction was chosen
* How the final answer was derived
* How the result was verified

---

## 📈 Progress

```text
LYKNCTF-2026   ██████████  1 writeup
UTECTF2026     ██████████  2 writeups
```

More writeups will be added as I solve more CTF challenges.

---

## 👨‍💻 Author

<div align="center">

**HuyVjpPro1806**

CTF Player • Reverse Engineering Learner • Blue Team / Forensics Path

[![GitHub](https://img.shields.io/badge/GitHub-HuyVjpPro1806-181717?style=for-the-badge\&logo=github)](https://github.com/HuyVjpPro1806)

</div>

---

<div align="center">

### ⭐ Thanks for visiting my CTF writeups!

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:22c55e,45:2563eb,100:020617&height=120&section=footer" />

</div>
