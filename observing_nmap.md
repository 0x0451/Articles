# How nmap scans work and look: 

## SYN Scan (-sS):

The SYN scan abuses the TCP handshake to perform "stealth" scans that are faster than the unprivileged connect scan. 

The typical TCP handshake is of course: 

- Client: SYN -> 
- Server: <- SYN/ACK
- Client: ACK -> 

This establishes a TCP connection over which communication can occur. The SYN scan works by cutting this process off half way so that the connection isn't fully formed. This is faster and considered "stealthy", although it's pretty easy to detect really.

### When the port is open

- client: SYN ->
- Server: <- SYN/ACK
- client: RST -> 

RST breaks the connection, and nmap calls it open because it got a SYN/ACK 

This will show action="close" (and pretty much *only* that, since no other communication occured) on a Fortigate which indicates that the client sent an RST packet (i.e. that the client broke the connection). This would indicate that the port is open. 

### If the port is "closed"

- client: SYN -> 
- Server: <- RST

This time, the server (really, the FW more likely) sends the RST to break the connection. In a fortigate, this would return action="server-rst". This action indicates that the server (or FW) reset the connection. 

### "Filtered"

Nmap's "filtered" result would result in

- Client: SYN -> 
- Server: (nothing)
- Client: SYN? (try again) -> 

There is no response from the host. On the inside, we'd probably see a variety of action types in a fortigate, like "dropped", etc, since there are a multitude of reasons why the scanner would receive no results. 

## Connect Scan (-sT or no flag without root/admin)

This scan works by simply connecting with a full TCP connection like so: 

- Client: SYN -> 
- Server: <- SYN/ACK
- Client: ACK -> 
- Server: <- DATA (e.g. like server banner, etc)
- Client RST -> 

This would look like any other sort of connection, with a communication channel being fully setup, and then ending in action=close. There is nothing in particular that sets it apart from any other sort of normal traffic, though. 
