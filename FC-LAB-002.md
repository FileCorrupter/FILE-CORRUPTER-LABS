# LAB 002 — Network Reconnaissance

**FILE CORRUPTER LLC | THINK. HACK. DEFEND.**

Difficulty: Beginner–Intermediate | Estimated duration: 45–60 minutes

## Mission
Learn to identify network assets, interpret exposed services, and recognize reconnaissance from a defender's perspective. Reconnaissance is information gathering, not proof of a vulnerability.

> Authorization: Perform every exercise exclusively on your own isolated virtual machines. Do not scan public addresses, school networks, employer networks, or third-party systems without explicit permission.

## Learning objectives
- Distinguish active and passive reconnaissance.
- Interpret IPv4 addressing, TCP and UDP, and port states.
- Discover lab hosts and identify services with Nmap.
- Observe discovery traffic in Wireshark and document findings.

## Environment
Use two disposable Linux VMs on the same virtual internal network, with no bridged adapter or route to external networks during exercises. Install Nmap and Wireshark on the analyst VM and Nginx on the target using your distribution's package manager before isolating them. Snapshot both VMs. Configure static addresses through your OS network settings; use no gateway on the internal interface.

```text
 Analyst VM                 Internal virtual switch          Target VM
 192.168.56.10/24  -------- [ FC-LAB-002 ] ----------------  192.168.56.20/24
 Nmap / Wireshark                                            Nginx / TCP 80
```

Addresses are examples; adapt them only to your dedicated lab. A /24 prefix covers 256 IPv4 addresses, normally including 254 usable host addresses. Check both VMs using `ip -br addr` and `ip route` before proceeding.

## 1. Reconnaissance fundamentals
Passive reconnaissance observes existing information or traffic without sending probes to the target. Active reconnaissance sends requests, for example ARP discovery or TCP connection attempts. Active probing can produce network and endpoint telemetry.

TCP is connection-oriented: a successful connection normally begins with SYN, SYN-ACK, ACK. UDP sends datagrams without that handshake. A port identifies a transport endpoint; TCP port 80 and UDP port 80 are distinct.

## 2. Establish ground truth
On the target VM, start the installed web server and inspect listeners:

```bash
sudo systemctl start nginx
sudo ss -lntup
```

Record which addresses and ports are listening. A loopback-only listener is not necessarily reachable from another VM.

## 3. Discover the authorized target
On the analyst VM:

```bash
ping -c 3 192.168.56.20
nmap -sn 192.168.56.20
ip neigh
```

`-sn` performs host discovery without port scanning. On a directly connected Ethernet network, Nmap can use ARP discovery when privileged. A missing ping reply alone does not establish that a host is offline. The neighbor table may contain previously resolved local addresses.

## 4. Identify TCP services
```bash
nmap -sT -p 22,80,443 192.168.56.20
nmap -sT -sV -p 80 192.168.56.20
curl -I http://192.168.56.20/
```

`-sT` uses operating-system TCP connections. `-p` limits the selected ports. `-sV` sends service-identification probes. HTTP headers provide additional clues but can be altered and should not be treated as definitive evidence of software version.

Illustrative results (not captured from an executed lab):

```text
PORT    STATE  SERVICE
22/tcp  closed ssh
80/tcp  open   http
443/tcp closed https
```

Actual results depend on listeners and firewall settings. Open means an application accepts connections; closed means the port responds but no application accepts connections; filtered means Nmap cannot determine the state because probes or responses may be blocked. A default service label based on port number is not proof of the actual application.

## 5. UDP comparison
```bash
sudo nmap -sU -p 53 192.168.56.20
```

A UDP service may not respond to a particular probe. ICMP port-unreachable messages can indicate a closed UDP port; no response may yield open|filtered. Do not interpret that ambiguous state as a confirmed open port.

## 6. Observe traffic
Start Wireshark on the analyst VM's internal lab interface before repeating the TCP scan. Use these display filters separately:

```text
arp
icmp
ip.addr == 192.168.56.20 && tcp
ip.addr == 192.168.56.20 && tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Identify the source and destination IPs, destination ports, and connection attempts. Compare successful handshakes with rejected connections. The SYN filter shows initial connection attempts, not conclusive evidence of malicious activity. For a passive exercise, observe existing lab traffic without generating additional probes.

## 7. Defensive validation
On the target, stop Nginx, then repeat the port-80 scan from the analyst VM.

```bash
sudo systemctl stop nginx
```

On the analyst:

```bash
nmap -sT -p 80 192.168.56.20
```

Compare the two observations. If no other process listens on TCP 80, expect closed or filtered depending on network policy.

Defenders should maintain asset inventories, disable unnecessary services, restrict access with firewalls and segmentation, monitor unusual connection patterns, and correlate network telemetry with authorized scanner schedules. A scan is not automatically an attack; context matters.

## 8. Record evidence
```bash
mkdir -p evidence
nmap -sT -p 22,80,443 -oN evidence/tcp-scan.txt 192.168.56.20
```

Record observation time, source, target, command, port state, validation method and recommended action. Keep real network identifiers, credentials and sensitive captures out of public repositories.

| Port | Observed state | Evidence | Recommended action |
|---|---|---|---|
| TCP 80 | Fill in | Scan and HTTP response | Keep only if required |
| TCP 22 | Fill in | Scan and listener check | Restrict administrative access |
| TCP 443 | Fill in | Scan and listener check | Enable only when required |

## 9. Knowledge assessment
1. How does passive reconnaissance differ from active probing?
2. Why does a failed ping not prove a host is offline?
3. What distinguishes closed from filtered TCP ports?
4. Why is an open port not necessarily a vulnerability?
5. Why can UDP results be ambiguous?

### Answer key
1. Passive observation does not directly probe the target; active discovery does.
2. ICMP may be blocked while other services remain reachable.
3. A closed port responds as unavailable; filtering prevents a definitive determination.
4. A legitimate, correctly configured and patched service may be intentionally exposed.
5. UDP has no connection handshake and services may silently discard unexpected requests.

## 10. Completion and cleanup
Save your lab-only findings, stop packet capture, stop the temporary web service if still running, and revert VM snapshots. Complete the lab when you can explain your observed port states and demonstrate how stopping a service changes exposure.

## References
- [Nmap reference guide](https://nmap.org/book/man.html)
- [Wireshark user guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- [MITRE ATT&CK: Network Service Discovery](https://attack.mitre.org/techniques/T1046/)

---
**FILE CORRUPTER LLC** — [filecorrupter.org](https://filecorrupter.org)

Next: LAB 003 — Digital Forensics: Hidden Data
