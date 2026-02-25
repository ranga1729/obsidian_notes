### **A. Cryptography & B. Numerical Questions**

- **Symmetric Encryption**: Uses a **single shared secret key** for both encryption and decryption.
    - **DES (Data Encryption Standard)**: Uses a **64-bit block size** and a **56-bit key**. It is now considered **insecure** because its small key space is vulnerable to brute-force attacks.
    - **AES (Advanced Encryption Standard)**: Uses a **128-bit block size** and supports key sizes of **128, 192, and 256 bits**. It is stronger and faster than Triple-DES.
- **Asymmetric (Public Key) Encryption**: Uses a **public key** for encryption/verification and a **private key** for decryption/signing.
    - **RSA**: A block cipher based on the difficulty of **factoring large prime numbers**. Encryption uses $C = M^e \mod n$, and decryption uses $M = C^d \mod n$.
    - **Diffie-Hellman (DH)**: A protocol for **secure key exchange**, not for general data encryption. It is based on the difficulty of the **discrete logarithm problem** in a finite field.
- **Hash Functions**: Condense arbitrary-sized messages into a **fixed-size fingerprint**; they are public and not keyed.
    - **MD5**: Produces a **128-bit hash** and is currently considered **vulnerable**.
    - **SHA-1**: Produces a **160-bit digest**.
- **Modes of Operation**:
    - **ECB (Electronic Codebook)**: Each block is encrypted independently; **repetitions in plaintext** may show in ciphertext, making it the weakest mode.
    - **CBC (Cipher Block Chaining)**: Each block is **chained** with the previous one; requires an **Initial Vector (IV)** and handles errors via propagation.
    - **CTR (Counter)**: Encrypts a **counter value** rather than feedback; it is ideal for **high-speed network encryption** because it allows for parallel processing.
- **Padding (PKCS5)**: Used when a message is not a multiple of the block size (e.g., 8 bytes); if the message is a perfect multiple, an **entire extra block** of padding is added.
- **Key Count Calculation**: In a symmetric network of $n$ users, the total number of keys required for all pairs to communicate is **$n(n-1)/2$**.
- **Euclidean Algorithm**: Used to efficiently find the **Greatest Common Divisor (GCD)** of two numbers, where $GCD(a, b) = GCD(b, a \mod b)$.

### **C. Security Fundamentals**

- **CIA Triad**:
    - **Confidentiality**: Assets are available only to **authorized parties**.
    - **Integrity**: Assets are modified only by authorized parties and in authorized ways; it implies data is **accurate and consistent**.
    - **Availability**: Systems provide **timely responses** and fair allocation of resources to users.
- **Additional Concepts**:
    - **Authenticity**: The ability to verify that a message or user is **who they claim to be**.
    - **Non-repudiation**: Prevents a party from **denying** a completed transaction or communication.
- **Controls**:
    - **Physical**: Isolation of equipment and access control to facilities.
    - **Technical (SW/HW)**: Internal software, OS controls, and special hardware devices.
    - **Administrative (Policies)**: Precise specifications, procedures, and organizational parameters.

### **D. Social Engineering & Human Factor**

- **Phishing**: Using **fake websites or emails** that mimic trusted sources to con users into supplying login credentials. **Spear-phishing** is a highly researched, targeted version of this attack.
- **User Awareness**: Security is incomplete without the user, who is **responsible** for the OS, applications, and data they manage.
- **Least Privilege Principle**: Users and applications should be granted only the **minimum level of access** necessary to perform their tasks.
- **Input Validation**: Defensive coding prevents vulnerabilities like **SQL Injection** (injecting commands into database queries), **XSS**, and **Buffer Overflow**.

### **E. System & Database Security**

- **System Security Planning**: Aimed at **maximizing security while minimizing costs** by determining requirements before initial deployment.
- **Security Maintenance**: A continuous process including **monitoring logs, patching software, and performing regular backups**.
- **Storage**: **Backup** involves making short-term copies for data recovery, while **Archive** is the long-term retention of data for legal or operational requirements.
- **Database Access Control**: Managed via SQL commands **GRANT** (bestow rights) and **REVOKE** (remove rights).
- **Inference**: The threat of users deducing **sensitive information** from multiple authorized queries of non-sensitive data.

### **F. Malware & Cybercrime**

- **Malware**: Programs inserted **covertly** to compromise the CIA of a victim's system.
    - **Virus**: Parasitic code that **attaches to executable content** and modifies other programs to replicate.
    - **Virus Life Cycle**: **Dormant phase** (idle), **Propagation phase** (replication), **Triggering phase** (activated by event), and **Execution phase** (performs function).
    - **Worm**: A standalone program that **actively seeks new machines** to infect via network connections.
    - **Backdoor**: A secret entry point that **bypasses normal security procedures**.
- **Cyber Law & Privacy**:
    - **Computer Crime vs. Cybercrime**: Computer crime involves computers as a tool or target; **cybercrime specifically connotes the use of networks**.
    - **Pseudonymity**: Allows a user to use a resource without disclosing their identity, but they **remain accountable** for that use.
    - **Sri Lanka Computer Crime Act (No. 24 of 2007)**: Identifies punishable offenses like unauthorized access and illegal interception; notably, owners are **obliged to surrender passwords or keys** to investigators.
    - _Note: The provided sources do not contain specific information regarding the pros and cons of GDPR._