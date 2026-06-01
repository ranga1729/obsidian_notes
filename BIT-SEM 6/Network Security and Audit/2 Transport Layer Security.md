![[Pasted image 20260516151218.png]]

Web traffic security can be implemented into,
1. Network Level(IP/IPSec)
2. Transport level(SSL or TLS)
3. Application level(Kerberos, S/MIME)

![[Pasted image 20260518101045.png]]
### What is TCP(Transmission Control Protocol) ?
![[Pasted image 20260518103519.png]]
IP works in the Network Layer and it doesn't care if packets gets out of order or not at all. 
TCP works in the Transport Layer and it wraps IP packets to ensure data actually makes send when it arrives. How ?
TCP is,
1. Connection Oriented
	TCP establish a formal connection between the sender and receiver before sending any packet. It does this using a 3-way handshake. (SYN --> SYN-ACK --> ACK).
2. Acknowledgement and Re-transmission
	Every time TCP sends a segment, it expects an Acknowledgement(ACK) back. If a time expires and no ACK arrives, TCP re-transmit it.
3. Ordered Delivery
	TCP assigns a sequence number to each packet so the receiver can re-assemble them in order. 
4. Flow Control(Sliding Window)
	Tell the sender how much data it can handle at that moment.
5. Congestion Control
	If TCP detects dropped packets, it assumes the network routers are maxed out, it backs off and slows down its transmission rate. (using algorithms like Tahoe, Reno, Cubic). This prevents the entire network from collapsing.

### How TCP and IP works together ?
IP packets wraps the TCP segment.
Steps,
1. The application Layer generates raw data and passes them to the Transport Layer.
2. Transport layer create cut data into manageable chunks and wraps the data in a TCP header(including source/destination ports, sequence numbers and checksums) and create a TCP segment. TCP then pass the segment to the Network layer.
	![[Pasted image 20260518103909.png]]
3. Network Layer wraps the entire segment in an IP header(containing source/destination addresses) and create a IP packet.
	![[Pasted image 20260518104223.png]]

### Where does TLS fit into the story ?
Right now, our data looks like,
	`[IP Header [TCP Header [Application Data]]]`
But the application data inside the packet is in plain text. This data is vulnerable to MITM attacks. 

TLS(Transport Layer Security) is the encryption layer that secures application data. It sits between the application layer(like HTTP) and the Transport layer(TCP). 
Ex: HTTPS means "HTTP running over TLS"

TLS doesn't change how TCP or IP works. It intercepts the raw application data. Instead it intercepts the raw application data and encrypt them to a cipher text and add its own tracking data and then hand them over to the TCP. 

Now the encapsulation structure is,
1. Application layer : Browser creates raw data. (ex: HTTP request)
2. TLS layer : Takes that data, encrypts it, add a TLS header. This is now a TLS record.
3. Transport layer(TCP): TCP takes the encrypted TLS record and wraps it in a TCP header. 
4. Network Layer(IP): IP takes the TCP segment and wraps it in a n IP header. 

**Protocol Handshake Order**
TCP uses 3-way handshake. TLS works as a second handshake as well. 
Steps,
1. The TCP Handshake(Establishing the pipe)
	This executes the `SYN --> SYN-ACK --> ACK`. In this process, no encryption has been involved. They just opening the communication channel. 
2. The TLS handshake
	Once the TCP connection is fully open, the TLS handshake begin inside that TCP channel.
	1. **Client Hello & Server Hello** : The browser and server agrees on which cryptographic algorithm(Cipher suite) they will use. 
	2. **Authentication** : The server send the digital certificate(signed by a CA).
	3. **Key Exchange** : Securely agree on a shared secret key using asymmetric cryptography. 
3. Secure Data transfer. 

Handshake protocol message types,

| Message type          | Parameters                                                    |
| --------------------- | ------------------------------------------------------------- |
| `hello_request`       | null                                                          |
| `client_hello`        | version, random, session id, cipher suite, compression method |
| `server_hello`        | version, random, session d, cipher suite, compression method  |
| `certificate`         | chain of X.509v3 certificates                                 |
| `server_key_exchange` | parameters, signature                                         |
| `certificate_request` | type, authorities                                             |
| `server_done`         | null                                                          |
| `certificate_verify`  | signature                                                     |
| `client_key_exchange` | parameters, signature                                         |
| finished              | hash value                                                    |


![[b83b75dbbf5b7e4be31c8000f91fc1a8.jpg]]

*****
### Transport Layer Security(TLS)
![[tls-in-osi.jpg]]
TLS is a general purpose service implemented as a set of protocols that rely on TCP. 

TLS is not a single protocol but two layers of protocols above TCP. 
![[Pasted image 20260516152021.png]]

Two important concepts in the TLS are,
1. **TLS session(Logical agreement)**
	When you first connect to a website, a full TLS handshake occurs. The server proves its identity, and both parties generate a master secret key. This entire cryptographic state is saved and assigned a unique **Session ID** (or a **Session Ticket**).
	This session contains the cipher suite, mater key, peer certification.
	Characteristics: abstract, cached, can last for hours/day.
2. **TLS connection(The actual pipeline)**
	A short-lived, operational, end-to-end communication channel that actually transfers the data.
	Every TLS connection runs on top of a specific TCP connection. If you close the browser tab/network drops, the TCP connection and the TLS connection gets destroyed. 
	Characteristics: Transient, concrete, tied directly to an active TCP socket. 
	This connection contains: security parameters derived by the session's master secret, unique initialization vectors, sequence numbers for just this specific burst of traffic. 

Since a TLS session can retain its data for days and the TLS connection is made with session data, a single TLS session can span multiple sequential connections. 
First connection will always perform a full handshake. subsequent connections will act as a connection resumption with the help of session data.

> [!NOTE] Access-Refresh Tokens
> This idea is somewhat similar to the Access token and Refresh token concept.

What TLS session state hold,
- Session identifier
- Peer certificate
- Compression method
- Cipher spec
- Mater secret
- Is resumable
- Server an d client random
- Server with MAC secret
- Client with MAC secret
- Server write key
- Client write key
- Initialization vectors
- Sequence numbers

### Internal Structures of the TLS
TLS is divided into two primary sub-layers,
1. Handshake layer(management/control)
2. Record layer(data plane)

#### 1. Handshake Layer
Core negotiator. When a client and server connect, this protocol exchange messages to,

**A. Handshake Protocol**
	- Negotiate cryptographic capabilities(choice of thew cipher suite, key exchange mechanism, hashing function)
	- Authenticate the identity of the server(and optionally the client) using digital certificates.
	- Securely exchange symmetric master key.

**B. Alert Protocol**
	The error-handling and session-closure mechanism. If anything goes wrong at any point , this alerts the peer. 
	Alerts have two levels of severity,
		- Warning(connection closure notice)
		- Fatal(an invalid certificate or decryption failure)
	If a fatal alert is triggered, the TLS connection is terminated immediately. 

**C. Change Cipher Spec Protocol**
	Deprecated mostly. Consist only a singe message. This acts as the boundary between unencrypted phase and the encrypted phase of the TLS protocol. 
	Once a client or server has calculated the new cryptographic keys using the handshake protocol, it send `Change cipher spec` message. 
	This essentially says, "Everything I send you after this exact byte will be encrypted using the keys and algorithms we just agreed upon."

#### 2. The Record Layer(The data plane)
Once the handshake layer agrees to encrypt data, the record layer handle the actual mechanism of taking application data and preparing it for transmission over TCP. 
It process outbound data through four stages,
	1. Fragmentation : Breakdown application data into manageable chunks called **"TLS records"**(16KB)
	2. Compression(optional/legacy) : Compresses the data to save bandwidth. 
	3. Authentication(MAC/AEAD) : Computes Message Authentication Code(MAC) or Authenticated Encryption with Associated Data(AEAD) algorithms. 
	4. Encryption: Encrypts the payload. 
	5. Append TLS record header
![[Pasted image 20260518150024.png]]

Once the record layer has appends a TLS record header, it hands it down to the Transport Layer(TCP).

TLS Record Header,
![[Pasted image 20260518145849.png]]

### Handshake protocol deep dive(TLS + TCP)
The complete handshake is in 4 phases,
1. Phase 1 - Establish security capabilities
2. Phase 2 -Server authentication and key exchange
3. Phase 3 - Client authentication and key exchange 
4. Phase 4 - Finish

#### Phase 1 - Establish security capabilities
![[Pasted image 20260518161201.png]]
During the handshake, Security details are negotiated during by `client_hello`(by the server) and `server_hello`(by the client) messages. 

Parameter of the client hello, 
- Version
- Random 
- Session ID
- Cipher Suite
- Compression method

Cipher suite is a set of cryptographic algorithms used to establish a secure network connection. 
![[Pasted image 20260518160119.png]]
Following key exchange methods are supported,
- RSA
- Fixed Diffie-Hellman
- Ephemeral Diffie-Hellman
- Anonymous Diffie-Hellman

Along with the key exchange method, it exchanges Cipher spec details including,
- Cipher Algorithm
- MAC algorithm
- Cipher Type
- Is Exportable
- Hash Size
- Key material
- IV size

#### Phase 2 - Server authentication
Server send certificate, key exchange and request certificate from the client. 
Server signals end of hello message phase. 
![[Pasted image 20260518161354.png]]

The `certificate` message is required to sent by the server for any agreed-on key exchange method except anonymous Diffie-Hellman. 

`server_key_exchange` message may be sent if it is required. It is not required in two instances,
- The server has sent a certificate with fixed Diffie-Hellman parameters OR
- RSA key exchange is to be used.

The `server_key_exchange` message is needed for the following,
- Anonymous Diffie-Hellman
- Ephemeral Diffie-Hellman
- RSA key exchange

An non-anonymous server(a server not using anonymous Diffie-Hellman) can request a certificate from the client by sending the `certificate_request` message. 
This message includes two params,
- certificate type
- certificate authority

The final `server_done` message is always required. 

#### Phase 3 - Client Authentication and Key Exchange
Client send certificate(if requested) and key exchange and a certificate(client's) verification.
![[Pasted image 20260518173447.png]]
#### Phase 4 - Finish
Send the `Change cipher suite` and finish handshake protocol.
![[Pasted image 20260518173843.png]]
Subsequent messages after this step are encrypted. 

****
### Heartbeat Protocol
In computer networking, a heartbeat protocol is a systematic mechanism used to monitor the availability of a network node or to maintain an active state over an idle communication channel. It operates as periodic, lightweight, query/response style. 

The heartbeat protocol directly runs on top of the **TLS Record Protocol** and consist of,
1. Heartbeat Request
2. Heartbeat Response

Purpose,
- Assure the sender that the recipient is still alive.
- Avoids closure of idle connection by a firewall.

In the TLS header, content type usually have 4 types,
- 20 - Change cipher Spec
- 21 - Alert
- 22 - Handshake
- 23 - Application Data
When the TLS is extended by the Heartbeat, it add a fifth message type 24 for the heartbeat messages. Payload of the Heartbeat response would be the exact copy of the payload sent in the Heartbeat request.


> [!NOTE] Heartbleed Attack
> Heartbeat protocol supposed to copy the payload in the request and send the it back in the heartbeat response's payload. This vulnerability occurred in the OpenSSL's implementation.
> 
> A malicious client would send a heartbeat request with a 1byte in the payload but length says its 64KB. Now vulnerable OpenSSL server copied the 1byte from the payload and buffer another 63,999 bytes from the system memory to fulfill the requested length. The server then sent this data in the heartbeat response.


SSL/TSL attacks can be targeted on, 
- Handshake protocol
- Record and application data protocols
- PKI(Public Key Infrastructure)

### TLSv1.2 vs TLSv1.3
Reason to moved to TLSv1.3 from 1.2,
- Removal of compression
	Compression algorithms exploit repeating patterns, an attacker who could inject text into a user's requests (like a malicious script running in a browser tab) could observe the exact byte-size changes of the resulting encrypted packets.
- Removal of CBC mode cipher and non-authenticated ciphers
	LSv1.2 permitted standard block ciphers(non-authenticated) operating in Cipher Block Chaining Mode(CBC) alongside MAC to achieve integrity and authenticity in two separate channels. 
	TLSv1.3 completely banned CBC and non-authenticated modes, mandating only AEDA(Authenticated Encryption with Associated Data) cipher like AES-GSM, ChaCha20. AEDA binds integrity and authenticity into a single atomic operation.
- Removal of Static RSA and Statis DH(Diffie-Hellman)
	TLS 1.2 allowed static RSA key transport, where the client encrypted a pre-master secret using the server’s public key.
	If attackers passively recorded years of encrypted network traffic and subsequently compromised or subpoenaed the server’s long-term private key, they could decrypt _all_ historically captured traffic. TLS 1.3 mandates **Perfect Forward Secrecy (PFS)**. It removes static RSA and static Diffie-Hellman (DH), allowing only Elliptic Curve Diffie-Hellman or Ephemeral Diffie-Hellman (ECDHE or DHE) key exchanges.
- Removal of 32-bit Timestamp in Client Hello
	The `Random` parameter in the TLS 1.2 `ClientHello` message consisted of 32 bytes, the first 4 of which were traditionally a Unix epoch timestamp.
	This structure leaked the client's local system clock, facilitating client tracking and device fingerprinting across different networks without providing any security benefit. Furthermore, it reduced the actual entropy of the random seed from 32 bytes to 28 bytes. TLS 1.3 mandates that all 32 bytes must be generated by a cryptographically secure pseudo-random number generator (CSPRNG).
- Deprecation of legacy RC4, MD5, SHA-224
	These algorithms have well-documented cryptographic weaknesses.
	TLS 1.3 strips them all out, relying entirely on the SHA-2 family (SHA-256/SHA-384) or HKDF constructions.
- Deprecation of inline re-negotiation
	TLS 1.2 allowed either peer to request a renegotiation mid-stream to refresh cryptographic keys or request client certificates.
	This introduced extreme complexity into the TLS state machine. It led to flaws like the **TLS Triple Handshake Attack**, where an attacker could spoof parameters across renegotiated sessions to inject data. TLS 1.3 eliminates inline renegotiation. If a server requires a key rotation, it sends a lightweight `KeyUpdate` post-handshake message, which updates the symmetric keys without restarting the entire protocol state machine.
- Removal of **Change Cipher Spec** protocol
	In TLS 1.2, the explicit 1-byte `Change Cipher Spec` message was required to inform the receiver that subsequent records would be encrypted.
	This required extra processing states and increased handshake overhead. In TLS 1.3, the handshake architecture is reorganized: encryption begins automatically immediately following the initial Hello exchange.

### SSL and TLS
SSL(Secure Socket Layer) was proprietary cryptographic protocol developed by Netscape Communication in the mid-1990s. It purpose was to secure transactions over the web(specifically for the Netscape Navigator browser) by introducing symmetric encryption, data integrity checks and public key authentication into the transport stack. 

Later it was handed over to the IETF(Internet Engineering Task Force) for open standardizations.
Then it was renamed to TLS(Transport Layer Security).

The term "SSL" became deeply entrenched in the tech industry during the dot-com boom so people still use the term SSL. Due to this reason Cryptographic certificates issued by Certificate Authorities (CAs) are still universally marketed as **"SSL Certificates."**

When a modern browser connects to a web server via HTTPS today, it executes a **TLS handshake**.

### HTTPS
Refers to the HTTP Combined with SSL(TLS) to implement secure communication between a web browser and a web server.
When HTTPS is used, following elements of the communication are encrypted,
- URL of the requested document
- Content of the document
- Content of the browser forms
- Cookies sent from browser to server and from server to browser. 
- Content of HTTP header. 

### SSH(Secure Shell)
**SSH (Secure Shell)** is a cryptographic network protocol operating at the **Application Layer (Layer 7)** that enables secure remote command-line login, remote command execution, and other secure network services over an unsecure network.

It runs on top of **TCP**, utilizing destination **port 22** by default. It was designed as a secure replacement for unencrypted legacy protocols like Telnet, rlogin, and `rsh`, which transmitted payloads and credentials in plaintext.

SSH is designed as three distinct, layered sub-protocols on top of TCP. 
1. Transport Layer Protocol
2. User Authentication Protocol
3. Connection Protocol
![[Pasted image 20260518194533.png]]

#### 1. Transport Layer Protocol
Foundation layer sitting directly on TCP. 
It establishes the cryptographically secure channel over an unreliable raw TCP connection and it is responsible for server authentication, algorithm negotiation, key exchange, and initiating encryption.
![[Pasted image 20260518195251.png]]
Steps,
1. Protocol Version exchange(Identification String)
	Once the TCP socket connection on port 22 is established, both the client and the server must state their protocol capabilities by transmitting a single line of ASCII text followed by a Carriage Return and Line Feed (CRLF).
	`SSH-ProtocolVersion-SoftwareVersion comments\r\n`
	Ex: SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13\r\n
2. Algorithm Negotiation(`SSH_MSG_KEXINIT`)
	The client and server must agree on which exact algorithms to use. Both parties construct and transmit an `SSH_MSG_KEXINIT` packet containing a list of supported primitives ordered by preference.
3. Key Exchange(KEX)
	Securely exchange(derive) keys using the selected key exchange algorithm. 
4. Server Authentication
	During the key exchange phase, the server must prove its identity to prevent a Man-in-the-Middle (MitM) interception. This occurs simultaneously with the conclusion of the KEX process.
5. Key Generation and Activation(`SSH_MSG_NEWKEYS`)
	At this stage, both endpoints possess the identical shared secret $K$ and Exchange Hash $H$, but they are not yet using them to encrypt traffic. Both sides pass $K$ and $H$ through a Key Derivation Function (KDF) or the negotiated hash algorithm. This derives distinct keys for asymmetric cryptographic operations:
	- Initial IV (Client-to-Server / Server-to-Client)
	- Encryption Key (Client-to-Server / Server-to-Client)
	- Integrity/MAC Key (Client-to-Server / Server-to-Client)
	Then each side transmits a single, unencrypted 1-byte packet: `SSH_MSG_NEWKEYS`.
	This message serves as a signal to the receiving side's internal state machine: _"Every packet I transmit after this exact byte will be encrypted and authenticated using the keys we just derived."_

#### 2. User Authentication Protocol
Execute immediately after the SSH Transport Layer Protocol has finished.(Now all traffic are encrypted). Purpose of this protocol is to authenticate client-side user to the server's OS. 

Steps,
1. The client sends and `SSH_MSG_SERVICE_REQUEST` packet explicitly naming the service name `ssh-userauth`
2. The server replies with an `SSH_MSG_SERVICE_ACCEPT` packet, initializing the user authentication state engine.
3. The client then begins submitting authentication attempts via `SSH_MSG_USERAUTH_REQUEST` packets. This request packet always contains three standard fields regardless of the method used:
	- Username
	- Service name
	- Method name

The server evaluates the request according to its local security policy. 
for every authentication request submitted, the server can only return one of two definitive packets:
- `SSH_MSG_USERAUTH_FAILURE` - If the credentials or signature fail validation.
- `SSH_MSG_USERAUTH_SUCCESS` - Sent when the authentication requirements are entirely satisfied.

The moment the client receives the `SSH_MSG_USERAUTH_SUCCESS` packet, the `SSH-USERAUTH` layer immediately terminates its state machine and tears itself down. Control of the encrypted transport channel is handed up to the final layer, the **SSH Connection Protocol (SSH-CONN)**, allowing terminal sessions and channels to open.

Authentication methods, 
- Public Key(Uses digital signatures)
- Password
- Host based

#### 3. Connection Protocol
Runs directly on top of the SSH Transport Layer Protocol. That connection is called the "Tunnel". 
Primary responsibility is "channel multiplexing". 
It takes the single, encrypted, bi-directional communication pipe provided by the transport layer and splits it into multiple logical, independent streams called **channels**.

Every channel is assigned an integer identifier by both the client and the server. When data is transmitted, the packet explicitly includes the recipient's channel ID number so the remote end knows exactly which application process should receive the payload.
Channels can be opened, used, and torn down dynamically at any point during an active SSH session without disrupting or terminating the underlying connection.

Types of channels,
1. Session - execute an interactive login shell, remote command
2. X11 - forward graphical X11 window packets
3. forwarded-tcpip - route raw TCP streams across the secure tunnel (Local).
4. direct-tcpip - route raw TCP streams across the secure tunnel (Remote).

![[Pasted image 20260518204150.png]]

SSH port forwarding is referred to as "SSH Tunneling".
types of port forwarding,
1. Local 
	Allows you to forward a port on your **local machine** (the client) to a port on a **remote machine** via the SSH server.
	ex: To access a remote MySQL database (port 3306) on your local machine at port 8080:
2. Remote
	AKA "Reverse Tunneling". 
	Forwards a port on the **remote SSH server** back to a port on your **local machine**.
	Use case: Sharing a web application you are developing locally with someone on the internet who can reach your public SSH server.
	ex: To allow others to see your local web server (port 80) by visiting port 8080 on the remote server

Port Forwarding with SSH provides the ability to convert any insecure TCP connection into a secure SSH connection.

##### Closing the Channel loop
When an application process concludes (for example, you type `exit` in your bash shell or an SFTP file transfer finishes), the channel is torn down cleanly:
1. The side terminating the process transmits an `SSH_MSG_CHANNEL_EOF` message, signaling that it will send no further data on this specific channel ID.
2. Once all remaining data in flight is drained and acknowledged, both sides exchange `SSH_MSG_CHANNEL_CLOSE` messages.
3. The specific channel IDs are freed up for reuse, while the underlying encrypted TCP pipe remains active to service any other open or future channels.