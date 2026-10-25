# Packet Lab

A hands-on packet analysis lab using Wireshark and `tcpdump`. This repository contains annotated network captures demonstrating core networking protocols and the OSI model in practice.

## Purpose
The goal of this lab is to move beyond theoretical networking and prove how traffic actually moves across interfaces. By capturing raw packets, we can observe how the Linux kernel routes traffic between the internal loopback interface (virtual network) and the external physical interfaces (LAN/Wi-Fi).

## Captures
Each capture has its own one-page analysis in the `/explanations` folder.
1. **[DNS Lookup](./explanations/dns-lookup.md):** Tracing the two-leg journey of a `dig` request. The capture shows the query moving from the `dig` process to the local resolver (`systemd-resolved`) over loopback, and then out to the LAN router to reach the internet. *(Completed)*
2. **[TLS 1.3 Handshake](./explanations/tls13-handshake.md):** Tracing the TCP three-way handshake and the TLS encryption negotiation for `example.com`, including SNI extraction and cipher suite selection. *(Completed)*
3. **[HTTP Reverse Proxy](./explanations/reverse-proxy-http.md):** Tracing an HTTP request through an Nginx proxy to a Python backend inside Docker, demonstrating network isolation, NAT routing, and header injection. *(Completed)*

## Repository Structure
*   `/captures`: The raw `.pcap` files recorded via `tcpdump`.
*   `/explanations`: Markdown files detailing the exact packet sequence, protocol details, and interpretation for each capture.
*   `/screenshots`: Visual evidence of the Wireshark analysis.
*   `/proxy-lab`: The Python, Nginx, and Docker Compose files used to generate the reverse proxy traffic.

## General Workflow
For every capture in this lab, the workflow is identical:
1. Run `tcpdump` with a BPF filter to record specific traffic to a `.pcap` file.
2. Generate the traffic using a command-line tool (e.g., `dig`, `curl`).
3. Stop the capture, open it in Wireshark, and apply a display filter to isolate the relevant packets from background OS noise.

## Security Considerations
Capturing and analyzing raw traffic is foundational for incident response and threat hunting.
*   **Visibility:** Without packet captures, defenders rely on high-level logs. Captures prove exactly what data left the machine and what protocols were used.
*   **Cleartext Risks (DNS):** Standard DNS uses cleartext UDP. An attacker on the local network (or the ISP) can read the queries, making DNS a prime target for passive sniffing and data exfiltration (DNS Tunneling). DNS-over-HTTPS (DoH) or DNS-over-TLS (DoT) is required to prevent this.
*   **Metadata Leakage (TLS):** While TLS 1.3 encrypts the actual web traffic, the Client Hello (containing the SNI) is sent in cleartext. This leaks the destination domain to anyone monitoring the wire, even when the payload is secure.
*   **Backend Isolation (HTTP Proxy):** A reverse proxy protects the backend by hiding its internal IP. However, the `X-Forwarded-For` header must be trusted only if the backend strictly accepts traffic from the proxy; otherwise, an attacker can spoof it to bypass IP-based controls.
*   **Protocol Mismatch (HTTP):** Translating between mismatched HTTP versions (like Python's HTTP/1.0 and Nginx's HTTP/1.1) without strict header sanitation can lead to HTTP Request Smuggling, allowing an attacker to poison the connection for the next user.
