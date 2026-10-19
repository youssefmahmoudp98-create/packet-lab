# Packet Lab

A hands-on packet analysis lab using Wireshark and `tcpdump`. This repository contains annotated network captures demonstrating core networking protocols and the OSI model in practice.

## Purpose
The goal of this lab is to move beyond theoretical networking and prove how traffic actually moves across interfaces. By capturing raw packets, we can observe how the Linux kernel routes traffic between the internal loopback interface (virtual network) and the external physical interfaces (LAN/Wi-Fi).

## Captures
Each capture has its own one-page analysis in the `/explanations` folder.
1. **[DNS Lookup](./explanations/dns-lookup.md):** Tracing the two-leg journey of a `dig` request. The capture shows the query moving from the `dig` process to the local resolver (`systemd-resolved`) over loopback, and then out to the LAN router to reach the internet. *(Completed)*
2. **TLS 1.3 Handshake:** *(Planned)*
3. **HTTP Reverse Proxy:** *(Planned)*

## Repository Structure
*   `/captures`: The raw `.pcap` files recorded via `tcpdump`.
*   `/explanations`: Markdown files detailing the exact packet sequence, protocol details, and interpretation for each capture.
*   `/screenshots`: Visual evidence of the Wireshark analysis.

## General Workflow
For every capture in this lab, the workflow is identical:
1. Run `tcpdump` with a BPF filter to record specific traffic to a `.pcap` file.
2. Generate the traffic using a command-line tool (e.g., `dig`, `curl`).
3. Stop the capture, open it in Wireshark, and apply a display filter to isolate the relevant packets from background OS noise.

## Security Considerations
Capturing and analyzing raw traffic is foundational for incident response and threat hunting.
*   **Visibility:** Without packet captures, defenders rely on high-level logs. Captures prove exactly what data left the machine.
*   **Attack Surface:** DNS is a common exfiltration vector (DNS Tunneling). Understanding the baseline structure of a legitimate DNS query (like the one captured here) is required to detect when an attacker is hiding data inside DNS requests.
*   **Encryption:** This DNS capture is cleartext UDP. An attacker on the local network (or the ISP) could read these queries. DNS-over-HTTPS (DoH) or DNS-over-TLS (DoT) is required to prevent this.
