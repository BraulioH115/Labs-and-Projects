Introduction:
This lab is focused on analyzing provided packet capture (PCAP) and system event logs to determine whether suspicious or malicious activity occurred on a home network. Wireshark Snort were used to examine traffic patterns, protocols, and host-based evidence. The main goal of this lab is to use any findings to conduct a story to determine whether this was malicious or not.
To complete the analysis, I approached it by first gathering the basic capture statistics, this includes the duration of the session, the total number of packets, and the total bytes that were transmitted. From there, I examined the traffic to identify the protocols that were used the most, what active (and common) hosts we had, and checking for any spikes on the graphs. Once we have that noted down, I then used Snort to determine if any alerts were triggered that could indicate any malicious activity. Finally, I used all the findings to correlate a story on the behavior coming from these logs and captures. 
Capture Statistics
•	Duration: 8 Minutes and 25 Seconds 
•	Total Packets: 2,449
•	Total Bytes: 811,157

Note: We are viewing the Lab1_Capture File first, after our findings from the security log, I utilized the Maybe Hacked PCAP to find evidence. 
Protocols Observed
Wireshark has a Protocol Hierarchy that can reveal multiple protocols including TCP, UDP, DNS, and HTTP. This tool can help us observe what type of malicious activity is going on (if any). TCP dominated the capture, having a percentage of 84.9, which we will see in our Snort Log why that is so high.  

Host & Network Information
•	Primary Host: 192.168.134.132 (suspected Windows machine).
•	Target Host: 192.168.134.129 (multiple SYN attempts observed from Snort Log).
•	Other Devices: Primarily LAN-based (192.168.x.x). No unrelated devices observed.
 


Traffic Spikes
A significant traffic spike was observed using the I/O graphs, I then changed the date format to correlate with where those spikes were. 
Figure 1: Maybe Hacked PCAP
 
Figure 2: Lab 01 Capture


After examining the spikes and logs, it showed that 192.168.134.132 sent multiple TCP SYN packets to 192.168.134.129, many of which were reset (RST, ACK). This suggests repeated failed connection attempts or scanning behavior.
 
 
Event Log Findings
- Security Log
The Windows Security Log recorded Event ID 4226 (TCP/IP concurrent connection limit reached) at 7/8/2017 2:56:46 PM. This event warns that the system attempted to open too many half-open TCP connections simultaneously. This directly correlates with the SYN flood-like activity observed in Wireshark, where numerous TCP connection attempts were made without successful completion.
 

- Application Log
The Windows Application Log also contained multiple warnings from the WinMgmt source (Event ID 63). These indicated that the provider HiPerfCooker_v1 was registered in the WMI namespace using the Local System account. Since this account is privileged, the warning noted that improper impersonation of user requests could lead to security violations. While this provider can be legitimate, WMI has been widely abused by malware as a persistence mechanism or to execute code stealthily. The repeated presence of Event 63 therefore adds to the suspicious nature of the host’s activity. I will provide a link explaining more about WMI and what an attacker can do with it. (References Tab)
 

ISP and External Access
Analysis of DNS queries and TLS handshake SNI fields showed that the host attempted to contact external services. 

 
 

While many connections resolved to raw IPs in the 64 and 65.x.x.x ranges, WHOIS lookups indicated these belonged to Microsoft using the printable text option. 
 
The analysis of the DNS filter reflects the normal behavior from the user using the internet to read emails. This suggests that much of the observed traffic could represent regular traffic and could be a false positive from an isolation point. However, given that we saw the multiple sign-on attempts from the security log, we should not completely disregard this evidence. 

Our Story
During the capture, Host 192.168.134.132 initiated numerous TCP SYN requests to Host 192.168.134.129. Many of these connections were refused or reset, leading to a spike in network traffic as seen on the graphs provided. At the same time, the Snort security log recorded Event ID 4226, confirming the TCP/IP stack hit its limit for concurrent connections.
Additionally, the host communicated with external IP addresses in the 64.x–66.x range, which resolved to Microsoft and Google infrastructure. DNS queries and WHOIs lookup confirmed access attempts to services within Microsoft (most likely email). This indicates account access activity during the capture.
Taken together, the evidence points to abnormal behavior involving mass connection attempts (possible scanning or misconfigured application) while also performing normal user actions like checking email accounts.
Conclusion
This lab demonstrated the importance of correlating network captures with host-based evidence. The key findings from our analysis included:
•	Excessive TCP SYN activity from Host 192.168.134.132, resulting in Event 4226.
•	Clear communication with ISP-related external resources, such as Microsoft.
•	Snort alerts that reinforced suspicious TCP activity.
Based on this evidence, the capture likely reflects suspicious behavior (possible scan) occurring alongside normal user account activity. Further investigation would be required to determine if this was malicious or a misconfigured application.
References
•	WHOis
•	WMI Red Canary
•	What is WMI?
•	RST and ACK packets on WIreshark
