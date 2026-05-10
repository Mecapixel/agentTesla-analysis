# Malware Analysis Report — AgentTesla Loader
**Analyst:** Meca D. | Pixora Inc.  
**Date:** May 9, 2026  
**Sample:** tmzZ.exe  
**Classification:** Trojan / Steganographic Loader / Infostealer Dropper  
**Severity:** High  

---

## 1. Executive Summary

`tmzZ.exe` is a .NET-based malware loader that disguises itself as a Doodle Jump mini-game. Upon execution, it silently extracts a hidden payload from an embedded bitmap image using a steganographic technique, loads it directly into memory without writing to disk, and executes it using .NET reflection. The inner payload is consistent with AgentTesla — a well-documented infostealer capable of credential theft, keylogging, clipboard capture, and data exfiltration.

---

## 2. Sample Information

| Field | Value |
|---|---|
| **Filename** | tmzZ.exe |
| **Full Path** | C:\Users\PonyBoy\Desktop\Malware-Samples\09783fa659755aba97256704aaf284df35d698ee9aff8cecd0eeeabe3a576a89.exe |
| **Timestamp** | 5/6/2026 10:48:22 AM |
| **Build Signature** | 69FB54B6 |
| **Framework** | .NET Framework 4.5 |
| **Entry Point** | DoodleJumpMini.Program.Main |
| **Payload Size** | 97,280 bytes extracted |
| **Campaign Tag** | IowsSzd |

---

## 3. Static Analysis

### 3.1 Fake Metadata (Cover Identity)

The binary presents itself as a legitimate game application:

| Field | Value |
|---|---|
| AssemblyTitle | DoodleJumpMini |
| AssemblyDescription | Doodle Jump Mini Game |
| AssemblyCompany | Antigravity |
| AssemblyProduct | DoodleJumpMini |
| AssemblyCopyright | Copyright © 2026 |

This metadata is entirely fabricated to evade casual inspection and deceive the victim into believing the file is safe.

### 3.2 Suspicious Imports

The following namespaces were imported — none of which are required for a simple game:

| Import | Purpose in Malware |
|---|---|
| `System.Reflection` | Load and execute assemblies from raw bytes in memory |
| `System.Runtime.InteropServices` | Direct Windows API access for low-level operations |
| `System.Runtime.CompilerServices` | Obfuscation support |

### 3.3 Class Structure

The assembly contains the following classes under `DoodleJumpMini`:

- `FormMenu` — **Contains the malicious loader logic**
- `FormPrincipal` — Game loop (legitimate game code)
- `LogicaSaltus` — Physics engine (legitimate game code)
- `FormPlacar` — Scoreboard (legitimate game code)
- `FormFinalizacao` — Game over screen (legitimate game code)
- `DadosDoodle` — Game data (legitimate game code)
- `Program` — Entry point, launches FormMenu

> **Note:** The game code is functional and real. It is used as cover to make the application appear legitimate while the malicious code executes silently in the background.

---

## 4. Loader Mechanism

### 4.1 Steganographic Payload Extraction

The core malicious method is `ArchiveSpoolDigest()` located in `FormMenu`. It is called during `InitializeComponent()` — meaning it executes **immediately when the application window loads**, before the user interacts with anything.

```
this.pf = FormMenu.ArchiveSpoolDigest(Resources.Tint, 97280, "RE-808", 5, 0.866, false);
```

**Parameters:**
| Parameter | Value | Purpose |
|---|---|---|
| `spoolReel` | `Resources.Tint` | Embedded bitmap image containing hidden payload |
| `digestQuota` | `97280` | Exact byte count to extract |
| `reelEpoch` | `"RE-808"` | Hardcoded key used in hash generation |
| `spindleTurns` | `5` | Iteration multiplier for math obfuscation |
| `inertiaCoeff` | `0.866` | Physics-like constant used as decryption parameter |
| `counterWind` | `false` | Direction flag for DCT coefficient processing |

**Extraction Process:**
1. Generates a hash key from `reelEpoch` using a custom xxHash-style algorithm with bitwise operations (`<<`, `>>`, `^`, `*`)
2. Runs the key through a pendulum physics simulation and DCT (Discrete Cosine Transform) — these are math obfuscation layers designed to waste analyst time and evade detection
3. Iterates through pixel coordinates of `Resources.Tint`
4. Reads the **R, G, B values** of each pixel sequentially
5. Appends each byte to `List<byte> pf` until `97,280` bytes are collected
6. Raises `HaltSignal` (a custom Exception class) to cleanly stop extraction

**Visual Evidence:**  
`Resources.Tint` appears as pure static/noise when viewed — a characteristic indicator of data hidden within pixel values. The resource is 100,072 bytes total, consistent with containing a 97,280 byte payload.

### 4.2 In-Memory Execution (Fileless)

After extraction, the payload is loaded and executed entirely in memory:

```csharp
Assembly assembly = AppDomain.CurrentDomain.Load(this.pf.ToArray());
Type type = assembly.GetExportedTypes()[0];
```

The malware uses `System.Activator` (obfuscated via Base64) to instantiate and run the payload:

```
Base64: U3lzdGVtLkFjdGl2YXRvcg==
Decoded: System.Activator
```

Method name `CreateInstance` is further obfuscated by constructing it at runtime:
```csharp
string text2 = Convert.ToChar(67).ToString() + "reateInstance";
// Result: "CreateInstance"
```

**This means the payload never touches disk** — it exists only in memory, making it invisible to file-based antivirus scanning.

---

## 5. Indicators of Compromise (IOCs)

| Type | Value |
|---|---|
| **File Hash (SHA-256)** | 09783fa659755aba97256704aaf284df35d698ee9aff8cecd0eeeabe3a576a89 |
| **Campaign Tag** | IowsSzd |
| **Hardcoded Key** | RE-808 |
| **Payload Size** | 97,280 bytes |
| **Embedded Resource** | Resources.Tint (100,072 bytes) |
| **Obfuscated String 1** | 496F7773 → Iows |
| **Obfuscated String 2** | 737A64 → Szd |
| **Base64 String** | U3lzdGVtLkFjdGl2YXRvcg== → System.Activator |

---

## 6. MITRE ATT&CK Mapping

| Technique ID | Technique Name | Observed Behavior |
|---|---|---|
| T1027 | Obfuscated Files or Information | Payload hidden in bitmap via steganography |
| T1027.002 | Software Packing | Payload embedded within carrier application |
| T1620 | Reflective Code Loading | Assembly loaded into memory via System.Reflection |
| T1036 | Masquerading | Binary disguised as DoodleJump game |
| T1140 | Deobfuscate/Decode Files or Information | Hex-encoded strings decoded at runtime |

---

## 7. Inner Payload — AgentTesla

The extracted payload is consistent with **AgentTesla**, a commodity infostealer. Based on published threat intelligence, AgentTesla is capable of:

- Credential theft from browsers, email clients, and VPN software
- Keylogging
- Clipboard capture
- Screenshot capture
- Data exfiltration via SMTP, FTP, or Telegram

> **Note:** The inner payload was not extracted or executed during this analysis. Attribution to AgentTesla is based on loader characteristics, campaign tag patterns, and threat intelligence from MITRE ATT&CK and public sandbox reports.

---

## 8. Analysis Environment

| Component | Details |
|---|---|
| **Host** | SHADYMECCA — HP Victus, i5-12450H, 32GB RAM, Windows 11 25H2 |
| **VM** | Windows 10 Pro (VMware Workstation) |
| **Network** | Host-Only — no internet access during analysis |
| **Tools Used** | dnSpy v6.5.1, CyberChef |
| **Sample Storage** | D:\VM\Malware-Tools\ (isolated external SSD) |

---

## 9. Disclaimer

This analysis was performed in an isolated, air-gapped virtual machine environment for educational and research purposes only. No malicious code was executed outside of the controlled lab environment. The sample was obtained legally for the purpose of malware research training.

---

*Pixora Inc. | Offensive Security Research*  
*github.com/Mecapixel*
