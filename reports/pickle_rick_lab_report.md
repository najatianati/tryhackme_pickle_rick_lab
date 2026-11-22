Penetration Testing Lab Report: Pickle Rick CTF
Platform: TryHackMe  
Room: Pickle Rick (Offensive Red Team CTF)  
Target Machine IP: `10.48.186.192`  
Assigned Attacker IP: `192.168.188.13` (OpenVPN)  
VPN Server: `ap-south-1-aynajatib5-regular`  
Student Account: `aynajatib5`  
Completion Status: Completed 
Date: November 21, 2026
---
Executive Summary
This report documents the step-by-step penetration testing methodology performed on the target server running the Pickle Rick CTF challenge on TryHackMe. The engagement encompassed network setup, service reconnaissance, web application enumeration, authentication bypass, remote command execution, command-filter evasion, system privilege escalation, and flag extraction. All three required ingredient flags were successfully recovered.
---
1. Challenge Room & Environment Setup
The target system operates within TryHackMe's isolated lab network. Remote access to the target host requires an active OpenVPN tunnel between the local security workstation and TryHackMe network infrastructure.
![Figure 1: TryHackMe Pickle Rick Challenge Room Overview](images/figure1_room_overview.png)  
Figure 1: TryHackMe Pickle Rick challenge room overview page.
![Figure 2: OpenVPN Client Connection Confirmation](images/figure2_vpn_connected.png)  
Figure 2: OpenVPN GUI connection popup confirming assigned local IP 192.168.188.13.
Network Connectivity Verification
Once the OpenVPN tunnel was active, network routing to target IP `10.48.186.192` was verified using an ICMP echo request from the Windows Command Prompt:
```cmd
C:\Users\IMPIAN>ping 10.48.186.192

Pinging 10.48.186.192 with 32 bytes of data:
Reply from 10.48.186.192: bytes=32 time=137ms TTL=62
Reply from 10.48.186.192: bytes=32 time=157ms TTL=62
Reply from 10.48.186.192: bytes=32 time=177ms TTL=62
Reply from 10.48.186.192: bytes=32 time=200ms TTL=62

Ping statistics for 10.48.186.192:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 137ms, Maximum = 200ms, Average = 167ms
```
![Figure 3: Target System Ping Verification](images/figure3_ping_target.png)  
Figure 3: Successful ICMP ping output confirming network reachability to 10.48.186.192.
---
2. Reconnaissance & Service Discovery
A network port scan was executed against the target IP using `Nmap` to detect active listening services, server versions, and host operating system signatures.
```cmd
C:\Users\IMPIAN>nmap -sC -sV 10.48.186.192
Starting Nmap 7.98 (https://nmap.org) at 2026-11-21 22:06 +0800
Nmap scan report for 10.48.186.192
Host is up (0.15s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 eb:46:a5:57:4a:d2:96:8c:e7:d8:53:9d:c3:9a:10:ac (RSA)
|   256 6d:06:8d:9c:7d:56:dc:ff:20:51:19:67:6c:d4:c3:99 (ECDSA)
|_  256 0a:f4:c4:70:d8:cc:68:a9:46:02:4c:6c:85:7d:f7:c3 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.32 seconds
```
![Figure 4: Nmap Port and Service Scan Output](images/figure4_nmap_scan.png)  
Figure 4: Nmap scan output identifying open SSH (port 22) and Apache Web Server (port 80).
Summary of Discovered Services
Port	Protocol	Service	Version Banner	Security Relevance
22	TCP	SSH	OpenSSH 8.2p1 Ubuntu 4ubuntu0.11	Secure shell access (requires private key or credentials).
80	TCP	HTTP	Apache httpd 2.4.41 (Ubuntu)	Web application hosting front-end portal; main initial entry vector.
---
3. Web Application Enumeration & Credential Gathering
Navigating to `http://10.48.186.192` in a browser rendered the Rick and Morty landing page titled "Help Morty!".
![Figure 5: Target Web Application Landing Page](images/figure5_web_homepage.png)  
Figure 5: Web application landing page hosted at http://10.48.186.192.
A. Inspecting HTML Page Source
Inspecting the HTML source code (`Ctrl + U`) uncovered a comment developer note left at the bottom of the page:
```html
<!--
Note to self, remember username!

Username: R1ckRul3s
-->
```
![Figure 6: HTML Source Code Inspection](images/figure6_page_source_username.png)  
Figure 6: HTML document source code revealing the hidden username comment `R1ckRul3s`.
B. Directory and Robots.txt Inspection
Visiting `http://10.48.186.192/robots.txt` exposed a plain-text string stored in the server's instruction file:
```text
Wubbalubbadubdub
```
![Figure 7: Robots.txt Secret Password File](images/figure7_robots_txt.png)  
Figure 7: Plain-text string `Wubbalubbadubdub` discovered inside http://10.48.186.192/robots.txt.
---
4. Initial Access & Web Shell Execution
With the username (`R1ckRul3s`) and password (`Wubbalubbadubdub`) retrieved, access was attempted at the portal login interface located at `http://10.48.186.192/login.php`.
![Figure 8: Web Portal Login Interface](images/figure8_login_page.png)  
Figure 8: Portal Login Page at http://10.48.186.192/login.php.
Submitting the credentials successfully authenticated the session and redirected to `portal.php`, which provides an interactive Command Panel capable of executing system commands as the `www-data` web server account.
---
5. Command Execution & Flag Retrieval
A. Ingredient 1: Web Root File (`Sup3rS3cretPickl3Ingred.txt`)
Executing `ls` inside the Command Panel listed files in `/var/www/html`:
`Sup3rS3cretPickl3Ingred.txt`, `assets`, `clue.txt`, `denied.php`, `index.html`, `login.php`, `portal.php`, and `robots.txt`.
Direct usage of `cat Sup3rS3cretPickl3Ingred.txt` was blocked by a server-side command filter. To bypass this control, `less` was executed:
```bash
less Sup3rS3cretPickl3Ingred.txt
```
![Figure 9: Ingredient 1 Execution Output](images/figure9_ingredient1_output.png)  
Figure 9: Command Panel output retrieving Ingredient 1 (`mr. meeseek hair`).
Ingredient 1 Flag: `mr. meeseek hair`
---
B. Ingredient 2: User Home Directory (`/home/rick`)
System directories were surveyed by listing contents of `/home`:
```bash
ls /home
# Output: rick, ubuntu
```
![Figure 10: Home Directory Listing](images/figure10_ls_home.png)  
Figure 10: Directory listing of /home showing user folders `rick` and `ubuntu`.
Listing files inside `/home/rick` identified `second ingredients`. Reading the file with escaped quotes:
```bash
less "/home/rick/second ingredients"
```
![Figure 11: Ingredient 2 Execution Output](images/figure11_ingredient2_output.png)  
Figure 11: Command Panel output retrieving Ingredient 2 (`1 jerry tear`).
Ingredient 2 Flag: `1 jerry tear`
---
6. System Privilege Escalation & Root Flag Extraction
To locate the final ingredient, administrative rights were evaluated. Inspecting `sudo` permissions revealed that the `www-data` user possessed passwordless `sudo` execution privileges across all commands (`(ALL : ALL) ALL`).
Listing `/root` with superuser privileges:
```bash
sudo ls /root
# Output: 3rd.txt
```
Reading the root file:
```bash
sudo less /root/3rd.txt
```
![Figure 12: Ingredient 3 Execution Output](images/figure12_ingredient3_output.png)  
Figure 12: Superuser execution output retrieving Ingredient 3 (`fleeb juice`).
Ingredient 3 Flag: `fleeb juice`
---
7. Results & Challenge Completion Summary
Task / Question	Captured Flag Value	Exploitation Methodology	Status
First Ingredient	`mr. meeseek hair`	Web Page Source Inspection + Command Evasion (`less`)	Completed
Second Ingredient	`1 jerry tear`	`robots.txt` Credential Harvesting + `/home/rick` Access	Completed
Third Ingredient	`fleeb juice`	Passwordless `sudo` Privilege Escalation + `/root` Read	Completed
---
Conclusion
The Pickle Rick CTF room was successfully compromised through systematic enumeration, credential extraction from public web files, web shell command filter bypass, and Linux privilege escalation to root. All flags were verified and submitted on TryHackMe.
