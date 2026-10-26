# Sanitized Capture Evidence

This file contains selected, manually sanitized details from the supplied lab outputs. Internal IP addresses and MAC addresses are omitted intentionally; the raw logs are not reproduced.

## tcpdump observations

- Capture interface: Ethernet interface (shown as `eth0` in the supplied log).
- Link type: Ethernet.
- Capture count: 10 packets.
- Filter: HTTP-related traffic on TCP port 80 involving the selected lab host.
- First packet: IPv4/TCP SYN from a client ephemeral port to server TCP port 80.
- Handshake: SYN → SYN-ACK → ACK.
- HTTP request: `GET / HTTP/1.1`, with `User-Agent: Wget/1.25.0`.
- HTTP response: `HTTP/1.1 200 OK`, served by Apache.
- Response body: `<html><body><h1>It works!</h1></body></html>`.
- Teardown: client FIN-ACK, server FIN-ACK, client ACK.

## curl observations

The supplied verbose curl transcript showed a connection to the lab web server on port 80, sent `GET / HTTP/1.1`, and received:

```http
HTTP/1.1 200 OK
Server: Apache
Content-Length: 45
Content-Type: text/html
```

It printed the same 45-byte HTML body documented above.

## Redaction and provenance

These excerpts summarize user-supplied course-lab files `tcpdump_log.txt` and `curl_log.txt`. They were transcribed for documentation; no new capture was made. Private lab endpoint addresses and hardware addresses are intentionally excluded. The original full logs should be retained privately unless the course permits their public redistribution.
