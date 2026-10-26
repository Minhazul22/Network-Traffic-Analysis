# Commands and Method

The following commands are drawn from the lab notes. Substitute the appropriate lab host and client addresses in your own authorized environment rather than copying private addresses into public documentation.

## 1. Connect to the lab host

```sh
ssh <lab-user>@<lab-server>
```

The session was established from inside the KLAB environment.

## 2. Inspect listening TCP sockets

```sh
ss -tlnp
```

This command was used to list listening TCP sockets and associated process information. The lab notes identified services including Apache on TCP port 80, GitLab on port 8095, and SSH on port 22.

## 3. Request the web page

```sh
wget http://<lab-server>/
curl -v http://<lab-server>/
```

Both tools returned a successful response in the lab. The response status was `HTTP/1.1 200 OK`, and the body was a small HTML page containing “It works!”.

The original lab command also included `-k` with curl while using an `http://` URL. That option is intended to skip certificate verification for TLS connections and is not needed for plain HTTP.

## 4. Capture HTTP-related packets

The lab capture used tcpdump with a packet count, numeric address/port output, verbose decoding, and hex/ASCII output. Its filter targeted TCP port 80 and one lab host.

A generalized form is:

```sh
sudo tcpdump -i <interface> -c 10 -n -vv -XX 'tcp port 80 and host <lab-host>'
```

- `-i`: choose the capture interface.
- `-c 10`: stop after capturing 10 packets.
- `-n`: avoid resolving addresses and ports to names.
- `-vv`: increase protocol decoding detail.
- `-XX`: display packet bytes in hexadecimal and ASCII, including the link-layer header.
- The filter limits output to traffic matching TCP port 80 and the selected host.

Only capture traffic on systems and networks where you are authorized to do so.

## 5. Interpret the output

1. Identify the SYN, SYN-ACK, and ACK that form the handshake.
2. Find the HTTP request line and relevant headers.
3. Identify the server's HTTP status and response payload.
4. Follow TCP flags through the final FIN/ACK packets.
5. Use the hexadecimal bytes to connect protocol fields with the human-readable tcpdump decode.

## Scope and limitations

This procedure records one short lab exchange. It does not constitute a broad vulnerability assessment, and it does not establish the security of the server beyond the observed HTTP transaction.
