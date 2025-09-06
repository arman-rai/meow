## **SMBMap Pentesting Cheatsheet**

### **1. Basics**

* **SMBMap**: Tool to enumerate and access Windows SMB shares without a GUI.
* **Purpose**: List shares, check permissions, download/upload files.
* **Ports**: Default SMB ports: `139`, `445`

---

### **2. Installation**

```bash
git clone https://github.com/ShawnDEvans/smbmap.git
cd smbmap
pip3 install -r requirements.txt
```

> Often pre-installed in pentest distros like Kali, Parrot, or in CTF boxes.

---

### **3. Enumeration**

* **List all shares (anonymous or with creds):**

```bash
smbmap -H <target>
```

* **Use credentials (domain\user):**

```bash
smbmap -H <target> -u <username> -p <password> -d <domain>
```

* **Check access/permissions for each share**

> Output shows: Read, Write, Create, Delete

* **Search for files across shares**:

```bash
smbmap -H <target> -u <user> -p <pass> -R -x 'passwords.txt'
```

> `-R` = recursive, `-x` = execute grep/search

---

### **4. Download / Upload**

* **Download file**:

```bash
smbmap -H <target> -u <user> -p <pass> -r <remote_path> -d <domain>
```

* **Upload file**:

```bash
smbmap -H <target> -u <user> -p <pass> --upload <local_file> <remote_path>
```

> Only works if share is writable

---

### **5. Quick Tips**

* Always start **anonymous check** → then use creds if available.
* SMB shares often reveal **passwords, configs, scripts, database backups**.
* Combine with **enum4linux** or **smbclient** for extra info.
* Look for **writeable shares** → possible persistence or payload upload.
* Use `-v` for verbose output: shows more detail about share access.

---

### **6. Example Workflow (CTF / pentest)**

```bash
# Step 1: Check open SMB ports
nmap -p 139,445 -sV <target>

# Step 2: Enumerate shares anonymously
smbmap -H <target>

# Step 3: Enumerate with credentials (if known)
smbmap -H <target> -u bob -p 'Passw0rd!' -d WORKGROUP

# Step 4: Recursive search for interesting files
smbmap -H <target> -u bob -p 'Passw0rd!' -R -x '.*\.txt$'

# Step 5: Download juicy files
smbmap -H <target> -u bob -p 'Passw0rd!' -r '/Finance/passwords.txt'
```

---
