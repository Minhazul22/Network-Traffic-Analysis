# Packet-by-Packet Analysis

This write-up is based on the supplied `tcpdump_log.txt` capture. The capture reported 10 packets and showed one short HTTP exchange. Endpoint addresses and MAC addresses are intentionally described by role rather than published in this public report.

## 1. Connection summary

| Capture stage | Direction | Interpretation |
|---|---|---|
| Packet 1 | Client → Server | TCP SYN begins a connection |
| Packet 2 | Server → Client | SYN-ACK acknowledges the SYN and offers the server's initial sequence number |
| Packet 3 | Client → Server | ACK completes the TCP three-way handshake |
| Packet 4 | Client → Server | HTTP `GET /` request with host and client headers |
| Packet 5 | Server → Client | TCP ACK acknowledges the request bytes |
| Packet 6 | Server → Client | HTTP `200 OK` response with headers and a 45-byte HTML body |
| Packet 7 | Client → Server | TCP ACK acknowledges the response |
| Packet 8 | Client → Server | FIN-ACK: client begins connection shutdown |
| Packet 9 | Server → Client | FIN-ACK: server closes its sending direction |
| Packet 10 | Client → Server | ACK completes the observed teardown sequence |

The exact packet ordering is the sequence shown in the supplied capture. The curl transcript was a separate command-line request; it is supporting evidence of the same successful resource response, not necessarily the very same TCP connection as this capture.

## 2. Packet 1: Ethernet II, IPv4, and TCP SYN

The first captured frame has an Ethernet header followed by an IPv4 header and a TCP header with TCP options.

### Ethernet II header

| Field | Interpretation |
|---|---|
| Destination MAC | Address of the next-hop destination on the local Ethernet segment; redacted here |
| Source MAC | Sender's interface address; redacted here |
| EtherType | `0x0800`, indicating IPv4 |

### IPv4 header

| Field | Observed value | Meaning |
|---|---|---|
| Version / IHL | `0x45` | IPv4, 20-byte base IP header |
| Total length | `60` bytes | Full IPv4 packet length, including the TCP header and options |
| Identification | `0xdf00` | IPv4 identification value for this packet |
| Flags / fragment offset | `0x4000` | Don't Fragment flag set; fragment offset is zero |
| TTL | `64` | Initial/remaining hop limit shown in this capture |
| Protocol | `6` | TCP |
| Source / destination IP | Redacted | Client endpoint to web-server endpoint |

### TCP header

| Field | Observed value | Meaning |
|---|---|---|
| Source port | `49716` | Client's ephemeral source port in this capture |
| Destination port | `80` | HTTP server port |
| Flags | `SYN` | Requests a TCP connection |
| Sequence number | `2245535166` | Client's initial sequence number as printed by tcpdump |
| Acknowledgement number | `0` | No acknowledgement number yet in this initial SYN |
| Window | `64240` | Advertised TCP receive window in this packet |
| Options | MSS, SACK permitted, timestamps, window scaling | TCP negotiation and performance-related parameters |

The TCP header contains options, so this first packet has a 40-byte TCP header rather than the minimum 20 bytes. With a 14-byte Ethernet II header and a 20-byte IPv4 header, the frame's captured packet payload length is consistent with a 60-byte IP packet containing no application data.

## 3. HTTP request

Packet 4 contains the request line:

```http
GET / HTTP/1.1
Host: [lab server]
User-Agent: Wget/1.25.0
Accept: */*
Accept-Encoding: identity
Connection: Keep-Alive
```

This identifies a request for the root path (`/`) using HTTP/1.1. The application data follows the TCP handshake, so the capture shows the transport connection being established before HTTP is exchanged.

## 4. HTTP response

Packet 6 contains the response status and headers, including:

```http
HTTP/1.1 200 OK
Server: Apache
Content-Length: 45
Content-Type: text/html
```

The response body shown in the capture is:

```html
<html><body><h1>It works!</h1></body></html>
```

The status `200 OK` indicates that the server successfully handled this request. The 45-byte content length matches the body shown in the capture.

## 5. TCP teardown

The last three packets shown in the capture are:

1. Client sends FIN-ACK.
2. Server sends FIN-ACK.
3. Client sends ACK.

This is consistent with an orderly TCP close for the connection in the capture.

## 6. Checksum note

Some server-originated packets in the supplied tcpdump output are marked with checksum text such as `incorrect -> ...`. Packet captures made on an endpoint can show apparent checksum errors when checksum offloading is involved, but the supplied output alone does not prove the cause. Treat this as an observation to investigate, not as a confirmed network fault.

## Conclusion

The capture demonstrates the expected sequence for a successful HTTP fetch over TCP: connection establishment, HTTP request, HTTP response, acknowledgements, and orderly connection teardown.
