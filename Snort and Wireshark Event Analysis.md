# Network Threat Detection Lab 

## Introduction

This lab focuses on analyzing provided packet capture (PCAP) and system event logs to determine whether suspicious or malicious activity occurred on a home network. **Wireshark** and **Snort** were used to examine traffic patterns, protocols, and host-based evidence. The goal was to construct a timeline or story from the findings to determine whether malicious behavior occurred.

The analysis process involved:
- Gathering basic capture statistics (duration, total packets, total bytes).
- Examining traffic for protocol distribution, active hosts, and traffic spikes.
- Running Snort to identify triggered alerts.
- Correlating all findings into a coherent narrative.

---

## Capture Statistics

- **Duration:** 8 minutes and 25 seconds  
- **Total Packets:** 2,449  
- **Total Bytes:** 811,157  
<img width="622" height="603" alt="A1" src="https://github.com/user-attachments/assets/4a3289c2-82ca-42fd-857d-b3d722ff9367" />

> *Note:* We first examined the `Lab1_Capture` file, then referenced the `Maybe Hacked PCAP` file to find supporting evidence.

---

## Protocols Observed

Wireshark's Protocol Hierarchy revealed multiple protocols including **TCP**, **UDP**, **DNS**, and **HTTP**.  
TCP dominated the capture, making up **84.9%** of the traffic — which Snort later helped explain.
<img width="941" height="597" alt="Traffic Of Protocols" src="https://github.com/user-attachments/assets/1c8825de-0138-4016-83a7-ec2674379112" />

---

## Host & Network Information

- **Primary Host:** `192.168.134.132` (suspected Windows machine)  
- **Target Host:** `192.168.134.129` (multiple SYN attempts observed via Snort)  
- **Other Devices:** LAN-based (`192.168.x.x`); no unrelated devices observed
<img width="1186" height="561" alt="IPV4 Endpoints" src="https://github.com/user-attachments/assets/4d493e7f-4fef-4d84-916f-ac5eecf7488b" />

---

## Traffic Spikes

Significant spikes were observed using Wireshark's I/O graphs. The date format was adjusted to correlate spike timestamps with log events.


**Figure 1:** Maybe Hacked PCAP  
<img width="306" height="268" alt="Maybe Hacked PCAP Graph" src="https://github.com/user-attachments/assets/5a465e49-e091-4b3f-9576-56754ccb0e26" />


**Figure 2:** Lab 01 Capture
<img width="852" height="671" alt="Data Transfer pikes" src="https://github.com/user-attachments/assets/049d41ce-8919-4bcf-a0b3-2c47521a7c2c" />

These spikes were caused by multiple TCP SYN packets sent from `192.168.134.132` to `192.168.134.129`, many of which were reset (RST, ACK), indicating failed connection attempts or scanning activity.
<img width="901" height="608" alt="Multiple TCP Login Attempts Found" src="https://github.com/user-attachments/assets/5990c4d0-3b6a-4681-8c9b-8df2da958766" />
<img width="901" height="608" alt="Multiple TCP Login Attempts Found" src="https://github.com/user-attachments/assets/db02f74c-7bca-47df-829f-0db44e603614" />

---

## Event Log Findings

### Security Log

- **Event ID 4226** (TCP/IP concurrent connection limit reached) recorded at `7/8/2017 2:56:46 PM`.
- Indicates the system attempted to open too many half-open TCP connections — correlating with observed SYN flood-like activity in Wireshark.
<img width="1033" height="661" alt="Security Log TCP Connect Attempts" src="https://github.com/user-attachments/assets/adb400ea-c080-44ac-9f4e-ee0551d7a6d0" />

### Application Log

- Multiple **Event ID 63** warnings from the `WinMgmt` source.
- Warned about the `HiPerfCooker_v1` WMI provider being registered under the `Local System` account.
- While possibly benign, WMI is often exploited by malware as a persistence or execution method.
- Repeated occurrences of this warning increased suspicion.
<img width="863" height="533" alt="Application Log" src="https://github.com/user-attachments/assets/a81d3bd2-00ca-48a5-8a31-bc8898ba9768" />

---

## ISP and External Access

- **DNS queries** and **TLS handshakes (SNI fields)** showed attempts to contact external services.
- WHOIS lookups confirmed many IPs in the `64.x.x.x` and `65.x.x.x` range belonged to **Microsoft**.
- DNS filter analysis suggested normal behavior (e.g. reading emails), but suspicious log activity suggests possible compromise.
<img width="1244" height="723" alt="dns filter" src="https://github.com/user-attachments/assets/6a33bb79-40f5-403e-ab63-09775179a4bb" />
<img width="1186" height="561" alt="IPV4 Endpoints" src="https://github.com/user-attachments/assets/ffd3aaad-a8a1-4114-8974-96742985a6e1" />

---

## Our Story

During the capture:
- Host `192.168.134.132` initiated numerous TCP SYN requests to Host `192.168.134.129`, most of which were refused/reset.
- This led to a spike in traffic.
- Snort logs confirmed **Event ID 4226**, indicating TCP/IP stack overload.
- The host also connected with external IPs belonging to **Microsoft** and **Google**.
- DNS and WHOIS confirmed access to Microsoft services — possibly email.

**Conclusion:** The behavior indicates mass connection attempts (possibly scanning or misconfiguration) coexisting with legitimate user activity.

---

## Conclusion

This lab highlighted the importance of correlating **network-level** data with **host-based** evidence.

### Key Findings:
- Excessive TCP SYN traffic from Host `192.168.134.132`, resulting in **Event 4226**.
- Communications with external IPs owned by **Microsoft**.
- **Snort alerts** confirmed potentially suspicious behavior.

> While the activity could be the result of misconfiguration, the evidence suggests scanning behavior worthy of deeper investigation.

---

## References

- [WHOIS Lookup](https://whois.arin.net/ui/?q=https%3A%2F%2Fi.imgur.com%2FZltISG3.png)  
- [WMI Red Canary](https://redcanary.com/threat-detection-report/techniques/windows-management-instrumentation/)  
- [What is WMI? (Microsoft)](https://learn.microsoft.com/en-us/answers/questions/3933779/what-is-this-wmi-warning-with-event-id-63-for)  
- [RST and ACK Packets in Wireshark](https://stackoverflow.com/questions/78046101/is-the-rst-ack-packet-in-wireshark-normal)  
