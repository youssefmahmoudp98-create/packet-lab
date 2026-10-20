# TLS 1.3 Handshake for `example.com`

## 1. Objective
To capture and analyze a TLS 1.3 handshake to observe how encryption is negotiated between a client and a server before application data is sent.

## 2. Environment
*   **Command:** `curl -v https://example.com`
*   **Capture Tool:** `sudo tcpdump -i any -nn -s 0 -w captures/tls13-handshake.pcap 'tcp port 443'`
*   **Filter applied in Wireshark:** `frame.number >= 66 && frame.number <= 72`

## 3. Packet Sequence
The capture contained 124 packets of background OS and browser traffic. The relevant sequence for our `curl` request is:
*   **Frame 66 (SYN):** The client (`192.168.1.5`) initiates a TCP connection to the server (`104.20.23.154`). Length: 80 bytes.
*   **Frame 67 (SYN, ACK):** The server acknowledges the request and responds. Length: 80 bytes.
*   **Frame 68 (ACK):** The client acknowledges the server. The TCP connection is established. Length: 72 bytes.
*   **Frame 69 (Client Hello):** The client sends a TLS Client Hello, requesting `example.com` via the SNI extension. Length: 1643 bytes.
*   *(Frames 70 and 71 are pure TCP ACKs ensuring the Client Hello was delivered reliably. They do not change the TLS state and are omitted for brevity).*
*   **Frame 72 (Server Hello):** The server responds, selecting the cipher suite and providing its certificate. Length: 1492 bytes.

## 4. Protocol Details
*   **Transport:** TCP on Port 443.
*   **TLS Version:** TLS 1.3 (Confirmed by the `supported_versions` extension, despite the legacy `0x0303` header).
*   **Server Name Indication (SNI):** `example.com` (visible in Frame 69).
*   **Cipher Suite:** `TLS_AES_256_GCM_SHA384` (selected by the server in Frame 72).

## 5. Interpretation
Unlike DNS (which uses connectionless UDP), TLS requires a reliable connection. The TCP three-way handshake (Frames 66-68) occurs first to establish the delivery path and negotiate parameters like window sizes. Once the path is open, the TLS handshake (Frames 69-72) negotiates the encryption keys. 

Interestingly, the Client Hello header claims to be `TLS 1.2` for backwards compatibility so that old network middleboxes (firewalls) don't drop the packet. The actual TLS 1.3 version is hidden inside the `supported_versions` extension. The server then selects one cipher suite out of the 30 offered by the client.

## 6. Security Considerations
The Client Hello (Frame 69) is sent in cleartext. This means a passive attacker on the local network (or the ISP) can read the packet and see exactly which website the client is connecting to via the SNI (`example.com`). While the actual web traffic is encrypted after the handshake, the SNI leaks metadata about the user's browsing habits. Modern privacy tools (like Encrypted Client Hello) aim to fix this gap.
