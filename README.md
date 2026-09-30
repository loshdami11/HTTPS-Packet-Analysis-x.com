# HTTPS-Packet-Analysis-x.com
![x.com](https://github.com/user-attachments/assets/1668c6ec-3c6f-4942-9083-6937bd4b8d22)



# Junior Network Analyst Report: HTTPS Packet Capture Analysis (x.com)

## Executive Summary
This project analyzes end-to-end network communication when opening a secure HTTPS connection to `x.com` (`172.66.0.227`). Network traffic was captured on a virtualized Kali Linux host using Wireshark to evaluate protocol interaction across the TCP/IP model.

---

## 1. TCP/IP and OSI Model Mapping

| TCP/IP Model Layer | Primary Function | Corresponding OSI Layer(s) | Key Protocols Involved |
| :--- | :--- | :--- | :--- |
| **4. Application** | User interface & protocol payload structure | Application, Presentation, Session (5–7) | HTTP/1.1, HTTP/2, TLS 1.3, DNS |
| **3. Transport** | End-to-end connection, segmentation, and reliability | Transport (4) | TCP (Dst Port 443), UDP (Port 53 / QUIC) |
| **2. Internet** | Packet addressing and logical routing | Network (3) | IPv4, IPv6, ICMP, ARP |
| **1. Network Access** | Physical frame delivery over local medium | Data Link, Physical (1–2) | Ethernet II (802.3), Wi-Fi (802.11) |

---

## 2. Packet Journey & Encapsulation Walkthrough

1. **Application Layer:** Web browser constructs HTTP `GET /` request and wraps it in TLS 1.3 encryption.
2. **Transport Layer:** Encrypted application data is segmented into a **TCP Segment** (Src Port: `37766`, Dst Port: `443`).
3. **Internet Layer:** The TCP segment is encapsulated into an **IP Packet** (Src IP: `192.168.136.133`, Dst IP: `172.66.0.227`).
4. **Network Access Layer:** The IP packet is encapsulated into an **Ethernet II Frame** (Src MAC: `00:0c:29:01:36:1a`, Dst MAC: `00:50:56:ee:fe:91`).
5. **Decapsulation at Destination:** The target server strips headers sequentially from Layer 1 up to Layer 4 to process the payload.

---

## 3. Host and Session Addressing Details

* **Client IPv4 Address:** `192.168.136.133`
* **Target IPv4 Address (x.com Edge CDN):** `172.66.0.227`
* **Gateway / DNS Resolver IPv4:** `192.168.136.2`
* **Client MAC Address:** `00:0c:29:01:36:1a` (VMware Virtual NIC)
* **Next-Hop Gateway MAC Address:** `00:50:56:ee:fe:91`
* **Source Port:** `37766`
* **Destination Port:** `443` (HTTPS)

---

## 4. Evidence & Screenshots

### Wireshark Packet Capture
![Wireshark Capture](https://github.com/user-attachments/assets/b257ba49-7475-4e78-a2d6-ffe7b746a79a)


### Frame 23 Header Details
![Frame Details](https://github.com/user-attachments/assets/bf6821cc-0e5c-4a49-90e1-4c61d18aa60e)


---

## 5. Captured Protocol Analysis Table

| Frame No. | Protocol | Source Address | Destination Address | Purpose / Summary |
| :--- | :--- | :--- | :--- | :--- |
| **19** | **DNS** | `192.168.136.133` | `192.168.136.2` | Standard query `A x.com` request. |
| **20** | **DNS** | `192.168.136.2` | `192.168.136.133` | Resolves `x.com` to `172.66.0.227`. |
| **23** | **TCP** | `192.168.136.133:37766` | `172.66.0.227:443` | **TCP SYN**: Connection handshake step 1. |
| **25** | **TCP** | `172.66.0.227:443` | `192.168.136.133:37766` | **TCP SYN-ACK**: Handshake step 2 response. |
| **26** | **TCP** | `192.168.136.133:37766` | `172.66.0.227:443` | **TCP ACK**: Handshake step 3 (Established). |
| **27** | **TLSv1.3** | `192.168.136.133:37766` | `172.66.0.227:443` | **Client Hello**: Negotiates ciphers & SNI (`x.com`). |
| **30** | **TLSv1.3** | `172.66.0.227:443` | `192.168.136.133:37766` | **Server Hello**: Server key exchange. |
| **32** | **TLSv1.3** | `172.66.0.227:443` | `192.168.136.133:37766` | **Application Data**: Encrypted web payload. |

---

## 6. TCP vs. UDP Comparison

| Feature | TCP | UDP |
| :--- | :--- | :--- |
| **Connection Type** | Connection-oriented (3-way Handshake) | Connectionless |
| **Reliability** | Guaranteed (ACKs, Retransmissions) | Unreliable / Best-effort |
| **Overhead** | Higher (20–60 byte header) | Lower (8 byte header) |
| **Use Cases** | Web Browsing (HTTPS), SSH, SFTP | DNS Queries, Video Streaming (QUIC) |

---

## 7. Failure Scenario & Troubleshooting Workflow

### Failure Scenario: DNS Resolution Timeout (`ERR_NAME_NOT_RESOLVED`)
* **Symptom:** Client cannot resolve `x.com` to an IP address (`172.66.0.227`), preventing TCP handshake initialization.

### Step-by-Step Diagnostic Commands
```bash
# 1. Verify Local Interface Status
ip a

# 2. Test Gateway ICMP Reachability
ping -c 2 192.168.136.2

# 3. Test DNS Resolution
nslookup x.com
nslookup x.com 8.8.8.8

# 4. Direct IP Port Binding Test
curl -I -v [https://172.66.0.227](https://172.66.0.227) -H "Host: x.com"

# 5. Restart DNS/Network Services
sudo systemctl restart NetworkManager

8. Technical References
Kurose, J. F., & Ross, K. W. (2021). Computer Networking: A Top-Down Approach (8th ed.). Pearson.

Rescorla, E. (2018). The Transport Layer Security (TLS) Protocol Version 1.3. RFC 8446.

Postel, J. (1981). Transmission Control Protocol. RFC 793.





