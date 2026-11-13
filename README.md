# NETWORKWALKS-EMMANUEL-B084-WK2-PM1-5-PEN-TESTING-REPORT
Week 2 footprinting and network scanning report
# PENETRATION TESTING REPORT

### Footprinting & Network Scanning Phases

**W2-PM-FINAL | CYBERSECURITY | NETWORKWALKS**

| Field | Detail |
| --- | --- |
| **Pentester Name (Cybersecurity Professional)** | Emmanuel-Osagie Esosa Ifeoluwa |
| **Program/Batch** | B084-Networkwalks |
| **Date** | 12 November 2026 |
| **Modules completed** | W2-PM1 (Multiple Kali Tools) and W2-PM5 (Zenmap Scanning) |
| **Client/Target** | 1. Networkwalks (networkwalks.com, the target assigned in the module instructions). 2. My own local lab network (10.0.0.0/24) |
| **Permission secured from client?** | Yes |
| **Phases covered** | Phase 1: Reconnaissance & Footprinting. Phase 2: Scanning & Network Discovery. Phase 3-5: In Progress |

---

## 1. Liability Disclaimer

I have performed these activities only on the systems & devices where I had secured written permission or the devices/systems that I own myself. All these materials are for education and research purpose only. Do not use anything from here to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries unauthorised access is a crime even when nothing is damaged.

## 2. Introduction

This report covers footprinting the networkwalks.com domain using multiple Kali Linux tools (W2-PM1) and scanning my own local lab network with Zenmap (W2-PM5). One module covers the footprinting phase and the other covers the scanning phase, so together they show how an attacker moves from gathering public information to mapping live hosts on a network. It is the Week 2 part of my internship program at Networkwalks.

All commands were run in a Kali Linux virtual machine (VirtualBox on a macOS host). Every step below describes the command used and the result I observed. The matching screenshots and raw output files are collected in section 8.

## 3. Tools Used

| Tool | Purpose |
| --- | --- |
| Kali Linux (VirtualBox VM) | Operating system used for all reconnaissance and scanning activities |
| whois | Find domain registration details (owner, dates, name servers) |
| whatweb | Fingerprint web technologies (server, CMS, plugins, IP) |
| nslookup | Resolve the domain name to its IP address using DNS |
| curl -I | Read the HTTP response headers of the website |
| wafw00f | Detect whether a Web Application Firewall protects the site |
| dnsrecon | Enumerate DNS records (NS, MX, SPF, TXT, SRV) |
| Zenmap (Nmap GUI) | Scan the local subnet to find live hosts, IPs and MAC addresses |
| ip a, ip neigh and ping | Local IP and MAC address identification, and checking a host's MAC and response |

## 4. Activities Performed

### 4.1 Footprinting & Reconnaissance

I performed reconnaissance against the networkwalks.com domain using six Kali Linux tools: whois, whatweb, nslookup, curl, wafw00f and dnsrecon. Each tool collected a different type of information, using only public sources.

First, I used **whois** to get the domain's registration details. The domain is registered with GoDaddy (created in 2019 and expiring in 2027), uses HostGator name servers, hides the owner's details behind a privacy service, and has DNSSEC unsigned.

I then used **whatweb** to identify the website's technologies. It showed an Apache server running **WordPress 7.1.4** with the **WordPress Download Manager 3.3.76** plugin and jQuery 3.7.1, and it exposed the contact email info@networkwalks.com.

Using **nslookup**, I resolved the domain to the IP address **192.232.216.135**, the same address whatweb reported.

I used **curl -I** to read the HTTP response headers. The HTTPS site returned HTTP/2 200 on Apache, and the headers exposed the WordPress REST API endpoint **/wp-json/**. A first run without https returned a 301 redirect to HTTPS.

Next, I used **wafw00f**, which showed that the site is protected by a **ModSecurity (SpiderLabs)** Web Application Firewall.

Finally, I used **dnsrecon** to enumerate DNS records. It showed HostGator name servers running BIND 9.16.23-RH, a mail server on the same IP as the website, an SPF record with soft fail (`~all`), and no SRV records.

### 4.2 Network Scanning with Zenmap

For the second activity, I used **Zenmap** (Nmap 7.99) on my Kali Linux virtual machine to perform network discovery on my own lab network. The practical required me to identify my local IP address and subnet, discover live hosts, record their IP and MAC addresses, and generate a network topology.

I first ran `ip a` to identify my local IP address and subnet. The lab interface (eth0) has the address **10.0.0.2/24**, so the subnet is **10.0.0.0/24**. The second interface (eth1) connects to my separate cybersecurity class lab network, which I use for other practice machines, so I did not scan it and my internship lab network stays untouched. I then started Zenmap with `sudo zenmap`, entered the subnet and selected the **Ping scan** profile, which runs `nmap -sn 10.0.0.0/24`. The scan checked 256 addresses and found **three live hosts**:

| IP address | MAC address | Notes |
| --- | --- | --- |
| 10.0.0.1 | 52:54:00:12:35:00 | QEMU virtual NIC, the network gateway |
| 10.0.0.2 | 08:00:27:cf:ed:39 | My own Kali machine. The MAC was taken from `ip a`, because a host does not show its own MAC in a scan |
| 10.0.0.3 | 52:54:00:12:35:00 | QEMU virtual NIC, responds to ping |

I repeated the scan from the terminal to save a text copy, and the result was the same. Two of the hosts, 10.0.0.1 and 10.0.0.3, report the same MAC address. After pinging 10.0.0.3 (3 of 3 replies), `ip neigh` showed the same MAC again. Two separate machines should not share a MAC address, so the virtual network device is probably answering for both. I recorded this as an observation and did not draw a firm conclusion.

After the scan, I opened the **Topology** tab in Zenmap, enabled the legend and saved the network topology as a PDF, as required by the practical. The graph shows localhost at the centre with the three hosts around it, all in green because a ping scan does not check ports. A ping scan only shows which hosts respond. It does not check ports, services or operating systems.

## 5. Risk Analysis / Impact

Based on the information collected during the footprinting and network scanning activities, I identified the following potential risks.

| # | Risk / Finding | Evidence / Observation | Potential Impact | Risk Level |
| --- | --- | --- | --- | --- |
| 1 | Web technology and version information exposed | WhatWeb identified WordPress 7.1.4 and WP Download Manager 3.3.76 | Attackers may use exposed version information to find software that needs further security review | **Medium** |
| 2 | Server IP address identifiable | nslookup resolved the domain to 192.232.216.135, also used by the mail server | Shows the network location of the web and mail services | **Low** |
| 3 | HTTP technical information exposed | curl -I showed caching headers, a plugin name in X-Redirect-By, and /wp-json/ in the Link header | May assist technology fingerprinting and further enumeration | **Low** |
| 4 | WAF technology identifiable | wafw00f identified ModSecurity (SpiderLabs) | Reveals information about the web application's security architecture | **Low** |
| 5 | DNS software version and infrastructure exposed | dnsrecon showed BIND 9.16.23-RH on the name servers, DNSSEC not signed, and an SPF soft fail (~all) | The version helps an attacker look up known issues. The name servers belong to the hosting provider | **Medium** |
| 6 | Public contact email exposed | whatweb showed info@networkwalks.com | Can be used for phishing or social engineering | **Low** |
| 7 | Multiple live hosts visible on local network | Zenmap found 3 live hosts, and two of them share one MAC address | Unknown devices, or a virtual device answering for several addresses, should be checked | **Low** |

**Risk level key:** Critical, Medium, Low

The risks above are observations from the footprinting and scanning exercises, not confirmed vulnerabilities.

The practical exercises primarily involved information gathering and host discovery. No exploitation or vulnerability validation was performed as part of these two modules.

Therefore, the presence of information such as a software version, IP address or DNS record does not by itself mean that the system is vulnerable. Further authorized security testing would be required to confirm any actual vulnerability.

## 6. Recommendations

Based on the observations from these activities, I recommend the following security improvements:

- **Review publicly exposed technology information.** Organizations should regularly review what information about their web technologies, CMS and plugins is publicly visible.
- **Keep software updated.** WordPress, plugins and other web technologies should be updated regularly and reviewed against current security advisories.
- **Review HTTP headers.** Response headers should be checked for unnecessary technical information, such as plugin names and the /wp-json/ link.
- **Review DNS records and software.** Records should be checked periodically so that only required information is public. The BIND version is shown by the hosting provider's name servers, so the owner should ask the provider to hide it. DNSSEC and a stricter SPF policy are worth considering.
- **Use a role address for public contact.** A shared role address is better than a single public email address.
- **Properly configure and monitor the WAF.** Keep the WAF (ModSecurity) enabled and tuned, since it already filters naive attacks.
- **Perform regular internal network discovery.** Organizations should periodically scan their own networks to identify active devices.
- **Investigate unknown devices.** Any unexpected device found during scanning should be verified, including cases like the shared MAC address seen here.
- **Maintain network documentation.** Network topology and device information should be documented and updated regularly.
- **Perform security testing with authorization.** Reconnaissance and scanning should only be performed against systems and networks where appropriate authorization has been provided.

## 7. Conclusion

During Week 2 of my Cybersecurity & Ethical Hacking internship, I completed practical activities covering footprinting, reconnaissance and network scanning.

In the footprinting activity, I used six Kali Linux tools to collect information about the target domain. I learned how whois provides domain information, whatweb identifies web technologies, nslookup resolves domain names, curl inspects HTTP headers, wafw00f identifies a WAF, and dnsrecon provides DNS information.

In the network scanning activity, I used Zenmap to identify my local network configuration and discover active hosts. I also collected IP and MAC address information and created a network topology.

The exercises showed me that information gathering is an important part of cybersecurity. Even before attempting to exploit a system, a security professional can learn a significant amount about an environment by carefully analyzing publicly available information and network responses.

I also learned that technical findings should be documented clearly. A good cybersecurity report explains what was performed, what was discovered, what the observation means, what risk it may create, and what can be done to reduce that risk. Finally, I learned that reconnaissance and scanning must always be performed within an authorized scope. These activities were completed as part of the assigned educational cybersecurity lab.

## 8. Evidences Collected

Screenshots are stored in [`screenshots/`](screenshots/) and the raw tool output in [`outputs_files/`](outputs_files/).

#### whois

<img width="1389" height="795" alt="1  whois A" src="https://github.com/user-attachments/assets/a74fbf98-76b4-4081-abff-0ad5cc78dafd" />


Raw output: [1_whois.txt](outputs_files/1_whois.txt) 



#### whatweb

<img width="1433" height="492" alt="what web new" src="https://github.com/user-attachments/assets/cfed1517-c1d9-4c5a-9a6a-18cff32703ed" />

**Note**: I typed an extra space in the save path, so the terminal printed “Is a directory” for the tee part. The command still ran and displayed its full output. I then moved 1_whois.txt and 2_whatweb.txt into the footprinting folder with mv, and ls confirms both files are there.

Raw output: [2_whatweb.txt](outputs_files/2_whatweb.txt)


#### nslookup

<img width="1422" height="275" alt="3 ns lookup, also has mov of who is" src="https://github.com/user-attachments/assets/f25cd8a7-ffa6-4a20-b57a-3522f3ab51db" />

Raw output: [3_nslookup.txt](outputs_files/3_nslookup.txt)

#### curl -I

I first ran `curl -I networkwalks.com` without `https://`, so the site answered with a 301 redirect to HTTPS. The task command uses `https://`, so I then ran `curl -I https://networkwalks.com`, which returned the full headers. Both results are shown.

**4a. Without https (301 redirect):** 

<img width="1433" height="549" alt="4  Curl without https" src="https://github.com/user-attachments/assets/bdbb93d1-92ab-416b-be14-83fde7017517" />


Raw output: [4_curl.txt](outputs_files/4_curl.txt)

**4b. With https (task command, HTTP/2 200):** 

<img width="1424" height="479" alt="4  Curl right" src="https://github.com/user-attachments/assets/e4200552-15b7-4eb3-bf57-1bd765044ecb" />


Raw output: [4b_curl_https.txt](outputs_files/4b_curl_https.txt)

#### wafw00f

<img width="1426" height="552" alt="5  wafwoof" src="https://github.com/user-attachments/assets/1203be7b-4bed-4fae-9d3c-8ea4e6af27fa" />


Raw output: [5_wafw00f.txt](outputs_files/5_wafw00f.txt)

#### dnsrecon

<img width="1431" height="522" alt="6  dns recon end" src="https://github.com/user-attachments/assets/d3d69f9c-7036-4bd6-a4e9-9bc1401bce6c" />


Raw output: [6_dnsrecon.txt](outputs_files/6_dnsrecon.txt)

#### Zenmap installed

<img width="1428" height="773" alt="zenmap install" src="https://github.com/user-attachments/assets/4ac5727e-d70b-4667-a0aa-8bf2812a72cf" />


#### local IP address and subnet

<img width="1433" height="812" alt="ip a" src="https://github.com/user-attachments/assets/c788188e-e0bf-47c1-89fa-1b868cbeba88" />


#### ping scan of the subnet

<img width="1420" height="779" alt="zenmap ip scan" src="https://github.com/user-attachments/assets/ae249066-2f21-4bdf-823f-17a8ba2204be" />


Raw output: [zenmap-ping-scan.txt](outputs_files/zenmap-ping-scan.txt) and [zenmap-ping-scan.xml](outputs_files/zenmap-ping-scan.xml)

#### MAC check

<img width="1426" height="509" alt="ip a and ip neigh" src="https://github.com/user-attachments/assets/d768440b-5a7c-4810-9515-28cfa7eeaf8c" />


#### Topology

<img width="1433" height="771" alt="zenmap graph topology" src="https://github.com/user-attachments/assets/5d673977-e512-428c-86ab-155d17f7ce77" />


Topology PDF: [zenmap-topology.pdf](outputs_files/zenmap-topology.pdf)

#### Author ####

**Emmanuel-Osagie Esosa Ifeoluwa**
LinkedIn: [https://www.linkedin.com/in/esosa-ifeoluwa-emmanuel-4b2364349](https://www.linkedin.com/in/esosa-ifeoluwa-emmanuel-4b2364349)

#### Project Information ####

**Program Name:** Cybersecurity program at Networkwalks | **Week:** 02 | **Repository:** Github


