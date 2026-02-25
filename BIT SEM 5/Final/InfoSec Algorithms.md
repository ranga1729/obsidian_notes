---
# 🔐 CRYPTOGRAPHIC ALGORITHMS — EXAM-FOCUSED FACT SHEETS
---
## 1️⃣ MD5 (Message Digest 5)
### 📌 Memorize These Points
- MD5 is a **hash function**
- Produces a **128-bit hash value**
- **Input size can be arbitrary**
- **Output size is fixed (128 bits)** regardless of input length
- Used for:
    - **Data integrity**
    - **Password hashing (historically)**
- **Not encryption** (irreversible)
- **Not secure** against collision attacks (important conceptual note)

📌 Typical Question Pattern:
> “MD5 generates a 128-bit hash from a 64-bit input” → **False**  
> (Hash size depends on algorithm, not input)

---
## 2️⃣ DES (Data Encryption Standard)

### 📌 Memorize These Points
- DES is a **symmetric-key block cipher**
- **Key size:** 56 bits
- **Block size:** 64 bits (8 bytes)
- **Considered insecure** (brute-force feasible)
- Uses **block-based encryption**
- When padding (PKCS5) is used:
    - Ciphertext size becomes **multiple of 8 bytes**

📌 Typical Question Pattern:
> “DES encrypts 30 bytes into 32 bytes using ECB + PKCS5” → **True**

---
## 3️⃣ AES (Advanced Encryption Standard)
### 📌 Memorize These Points
- AES is a **symmetric-key block cipher**
- **Block size:** **128 bits (fixed)**
- **Key sizes:** 128, 192, 256 bits
- Plaintext size = Ciphertext size **(in CTR mode)**
- Secure and **recommended standard**
- Faster than DES

📌 Typical Question Pattern:
> “AES encrypts 30 bytes into 32 bytes in CTR mode” → **False**  
> (CTR does NOT expand data)

---
## 4️⃣ RSA (Rivest–Shamir–Adleman)
### 📌 Memorize These Points
- RSA is an **asymmetric cryptographic algorithm**
- Uses **public key (e, n)** and **private key (d, n)**
- **Encryption:**  
$$
    C = M^e \bmod n
$$
- **Decryption:**  
$$ 
    M = C^d \bmod n  
$$
    
- **Digital Signature:**  
$$
    S = M^d \bmod n  
$$
- **Signature Verification:**  
 $$
     M = S^e \bmod n  
 $$
- Key generation rule:  
$$
    e \times d \equiv 1 \pmod{(p-1)(q-1)}  
$$

📌 Typical Question Patterns:  
✔ Validate if `d` is correct  
✔ Compute encryption/signature  
✔ Identify **false RSA statements**

---

## 5️⃣ Diffie–Hellman Key Exchange
### 📌 Memorize These Points
- Used for **key agreement**, not encryption
- Public parameters: **g, n**
- Private keys: **x, y**
- Public keys:  
$$
X = g^x \bmod n  
$$
$$
Y = g^y \bmod n
$$
- Shared session key:  
$$ 
    K = Y^x \bmod n = X^y \bmod n  
$$
- Both sides must compute **same session key**

📌 Typical Question Pattern:
> “The session key is X” → **True/False + Calculation**

---

## 6️⃣ Modes of Operation (ECB, CBC, CTR)
### 📌 Memorize These Points
### 🔹 ECB (Electronic Code Book)
- **Insecure**
- Same plaintext → same ciphertext
- **Prone to cryptanalysis**
- Requires padding
### 🔹 CBC (Cipher Block Chaining)
- More secure than ECB
- Encryption is **sequential**
- Slower than CTR
### 🔹 CTR (Counter Mode)
- Encrypts counters, not plaintext
- **Plaintext length = Ciphertext length**
- **Fast** (parallelizable)
- No padding required

📌 Typical Question Pattern:
> “CTR is faster than CBC” → **True**

---

## 7️⃣ Hash vs Encryption (VERY COMMON TRAP)

|Feature|Hash|Encryption|
|---|---|---|
|Reversible|❌ No|✅ Yes|
|Key used|❌ No|✅ Yes|
|Integrity|✅ Yes|❌ No|
|Confidentiality|❌ No|✅ Yes|

---
# 🔐 Cryptographic Algorithm Comparison Table

## 1️⃣ High-Level Classification

|Algorithm|Type|Key Used|Reversible|Primary Purpose|
|---|---|---|---|---|
|**MD5**|Hash|None|❌ No|Integrity|
|**SHA-1**|Hash|None|❌ No|Integrity|
|**DES**|Symmetric|Same key|✅ Yes|Confidentiality|
|**AES**|Symmetric|Same key|✅ Yes|Confidentiality|
|**RSA**|Asymmetric|Public / Private|✅ Yes|Encryption, Signatures|
|**Diffie-Hellman**|Key Agreement|Public / Private|❌ No|Key Exchange|

---

## 2️⃣ Core Technical Facts (🔥 Frequently Tested)

|Algorithm|Block Size|Key Size|Output Size|Secure?|
|---|---|---|---|---|
|**MD5**|N/A|N/A|**128 bits**|❌ No|
|**SHA-1**|N/A|N/A|**160 bits**|❌ Weak|
|**DES**|**64 bits**|**56 bits**|Same as input|❌ No|
|**AES**|**128 bits**|128 / 192 / 256 bits|Same as input|✅ Yes|
|**RSA**|N/A|Variable|Variable|⚠️ Depends|
|**Diffie-Hellman**|N/A|Variable|Session key|⚠️ Depends|

📌 **Exam Trap:**  
Hash output size **never depends on input length**.

---

## 3️⃣ Modes of Operation Comparison (VERY COMMON)

|Mode|Padding|Secure?|Speed|Key Feature|
|---|---|---|---|---|
|**ECB**|Yes|❌ No|Fast|Pattern leakage|
|**CBC**|Yes|✅ Yes|Slow|Chaining|
|**CTR**|No|✅ Yes|**Fastest**|Parallelizable|

📌 **Exam Gold Statements**

- ECB → **Insecure**
- CTR → **Plaintext length = Ciphertext length**
- CTR faster than CBC → **True**