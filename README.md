# Samba AD DC & Multi-OS Client Integration Lab (Linux Mint & Windows 10/11)

A robust, hands-on enterprise lab documentation detailing the deployment, configuration, and troubleshooting of a **Samba Active Directory Domain Controller (AD DC)** in a strictly isolated, offline environment—integrated with both Linux Mint and Windows 10/11 clients.

---

## 🛠️ Architecture & Network Topology

* **Environment:** Isolated, offline lab network (no internet gateway)
* **Domain Name:** `SZERVEZET.LOCAL`
* **Samba AD DC Server (`dc01`):**
  * **IP Address:** `192.168.1.10`
  * **Role:** Active Directory Domain Controller, Internal DNS Server, Local NTP Time Source
* **Linux Mint Client:**
  * **IP Address:** `192.168.1.60` 
  * **Integration:** SSSD, Kerberos, and Realmd configuration for domain user authentication.
* **Windows 10/11 Client:**
  * **IP Address:** `192.168.1.50`
  * **Subnet Mask:** `255.255.255.0`
  * **Primary DNS:** `192.168.1.10` (Gateway-free setup)

---

## 📋 Implementation & Troubleshooting Steps

### 1. Linux Mint Client Integration (SSSD & Kerberos)
* Configured local resolver and Kerberos realm settings (`/etc/krb5.conf`).
* Joined the Linux Mint client to the `SZERVEZET.LOCAL` domain using `realmd` and configured `SSSD` for system-level domain user authentication and local home directory creation.

### 2. Network & Static DNS Configuration (Windows Client)
Configured static IPv4 addressing and enforced the internal Samba AD DC as the exclusive DNS server via the classic control panel (`ncpa.cpl`).
* **Verification Command:**
  
  ```cmd
  ipconfig /all
  ```
 
###  3. Resolving Name Resolution & NAT Conflicts
To overcome initial VirtualBox NAT adapter caching issues (10.0.2.15), the server-side DNS records and host files were explicitly mapped:

* **Server Host File Configuration** 
```bash
(/etc/hosts):
192.168.1.10 dc01.szervezet.local dc01
```
* **Updating Samba DNS Records (samba-tool):**
```bash
sudo samba-tool dns update 127.0.0.1 SZERVEZET.LOCAL dc01 A 10.0.2.15 192.168.1.10 -U administrator
sudo samba-tool dns delete 127.0.0.1 SZERVEZET.LOCAL @ A 10.0.2.15 -U administrator
sudo samba-tool dns add 127.0.0.1 SZERVEZET.LOCAL @ A 192.168.1.10 -U administrator
```

* **Client Cache Flush & Verification:**
```bash
ipconfig /flushdns
nslookup SZERVEZET.LOCAL
```
### 4. Joining the Windows Client to the Domain
Executed the workstation integration into the active directory structure.
Execution Steps:

* **Opened System Properties via sysdm.cpl**
* **Changed computer membership to Domain: SZERVEZET.LOCAL**
* **Authenticated using domain administrator credentials (administrator)**
* **Successfully joined and restarted the workstation**

### 5. Server-Side Health Check & NTP Verification
Performed database integrity checks and monitored time synchronization required for Kerberos authentication:

* **Verification Commands:**
```bash
sudo samba-tool drs showrepl
sudo samba-tool dbcheck
timedatectl 
```
* **Diagnostic Results:**
Database Check: 0 errors found across 286 objects.
NTP Service: Active local time provider.

### 🚀 Key Takeaways & Skills Demonstrated
  * **Manual, engineering-focused deployment of open-source Directory Services.
  * **Advanced DNS troubleshooting and record manipulation via samba-tool.
  * **Cross-platform heterogeneous domain integration (Linux SSSD & Windows NetJoin).
  * **Strict offline network management and replication health auditing.

