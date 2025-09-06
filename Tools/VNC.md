## **VNC Pentesting Cheatsheet**

### **1. Basics**

- **VNC (Virtual Network Computing)**: Remote desktop protocol (RFB – Remote FrameBuffer).
- **Ports**: Default `5900 + display number`. Common: 5900, 5901…
- **Auth types**:
    - **Password** (6-8 chars max, DES-based)
    - None (rare, unsecured)
- **Clients/Tools**:
    - `vncviewer` (RealVNC, TigerVNC)
    - `Remmina` (Linux GUI)
    - `Nmap` scripts

---

### **2. Enumeration**
- **Check if VNC port is open**: 

```bash
nmap -p 5900-5910 -sV -sT <target>
```

- **VNC-specific NSE scripts**:

```bash
nmap -p 5900 --script vnc-info <target>
```

> Returns supported versions, password authentication, desktop name, etc.

- **Brute-force password** (if allowed in engagement/CTF):

```bash
hydra -P passwords.txt -t 4 <target> vnc
```

- **Check banner manually**:

```bash
nc <target> 5900
```

> VNC responds with version banner like `RFB 003.008`.

---

### **3. Exploitation / Attack**

- **Default / weak passwords**: Try `vncpasswd`, `123456`, `password`.
- **Password cracking offline**:
    - Grab VNC password hash via tool: `vncpwd.py` or `vncpasswd-dump`
    - Crack with `hashcat`:

```bash
hashcat -m 5900 hash.txt wordlist.txt
```

- **Man-in-the-middle / Sniffing**: VNC is often **unencrypted** (unless via TLS). Use Wireshark or `Bettercap` to capture passwords.
- **VNC over SSH tunnel** (for pivoting / bypassing firewall):

```bash
ssh -L 5900:localhost:5900 user@target
vncviewer localhost:5900
```

---

### **4. Post-Exploitation**

- Full desktop control → dump sensitive info:
    - `cmd.exe`, `powershell` commands
    - Keylogger tools (if in scope)
    - Lateral movement inside network

---

### **5. Quick Notes / Tips**

- VNC password max length **8 chars**, DES-encrypted → weak brute-force possible.
- TigerVNC allows **encrypted passwords**, but many servers still use plaintext.
- Always **enumerate first**, don’t just brute-force. Use `vnc-info` NSE script for version and auth type.
- Consider **proxy or pivoting** if port blocked externally.

---