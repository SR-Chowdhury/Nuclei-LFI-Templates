# Custom Nuclei LFI Templates

This repository contains custom Nuclei templates designed to detect Local File Inclusion (LFI) vulnerabilities on both Linux and Windows servers.

## 📖 What is LFI?
**Local File Inclusion (LFI)** is a vulnerability that occurs when a web application allows an attacker to include files on the server through user-controlled input (usually URL parameters). If not properly validated, an attacker can use path traversal techniques (e.g., `../../../../`) to read sensitive system files outside the web root.

### Server Types Target
The templates in this repository are designed to detect LFI on two main server environments:
1. **Linux Servers:** Attempts to read `/etc/passwd` (a file that contains user account information).
2. **Windows Servers:** Attempts to read `C:\Windows\win.ini` or `boot.ini` (legacy configuration files).

## 🛠️ Included Templates

### 1. `LFI-template-1.yaml` (Hardcoded Payloads)
This template contains a predefined set of highly effective path traversal payloads (including URL encoding and basic bypasses). It is fast, lightweight, and great for quick scans.

### 2. `LFI-template-2.yaml` (File-based Payloads)
This template reads payloads from a separate text file (`payloads/lfi-payloads.txt`). This is useful when you want to test hundreds of payloads (e.g., from a wordlist) without making the YAML file messy.

## 🚀 How to Use

### Prerequisites
- Install [Nuclei](https://github.com/projectdiscovery/nuclei) (v3.0.0 or higher is recommended).
- Clone this repository:
  ```bash
  git clone https://github.com/SR-Chowdhury/Nuclei-LFI-Templates
  cd Nuclei-LFI-Templates

## ⚠️ Disclaimer & Legal Notice

The templates and payloads in this repository are intended for **educational purposes and authorized security testing only**. 

- **Do not** use these templates against any target without obtaining explicit, written permission from the owner.
- Unauthorized scanning, hacking, or exploiting is **illegal** and strictly prohibited.
- The author assumes **no liability** and is not responsible for any misuse or damage caused by these tools.

Always practice in safe, isolated environments such as DVWA, TryHackMe, Hack The Box, or your own local lab.