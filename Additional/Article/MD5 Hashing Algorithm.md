##### What is hashing ?
![[Pasted image 20250906191317.png]]
Hashing is the technique in computer science and cryptography where data of any size converted to a fixed value(a hash/digest/checksum) using a mathematical algorithm.
#### Ronald Rivest's journey with cryptography
![[Pasted image 20250906192303.png]]
Ronald Rivest, a MIT professor and co-inventor of famous RSA algorithm, is a key figure in cryptography. He developed a series of "Message Digest"(MD) algorithms to provide a fast and efficient message digests for digital signature applications. 

MD2(1989)
Rivest's first major hashing algorithm was published in 1989. It was designed for resource-constrained 8-bit processors, which were very common at the time and became a standard among RSA's public-key cryptosystems for compressing large messages. 
Even though its was an early success, it's now considered insecure. 

MD4(1990)
MD4 is a radical redesign of it's predecessor. 
The main focus of the creation of MD4 was to have an incredibly fast hashing algorithm for 32-bit machines, which were becoming the new standard in computing.
This new design was simplified with some trade-offs for performance. 
This  is the same reason led it to have some severe security vulnerabilities. 
Within a year, cryptographers found weaknesses that could be exploited to find "collisions" much faster than brute force. 
(Collisions : Two different inputs that produce the same hash output) 

MD5(1991)
MD5 was developed as a direct response to the cryptographic vulnerabilities found in MD4 algorithm. The main focus of the new MD5 design was to retain the speed of MD4, while addressing it's security flaws. 
As a solution, Rivest added an extra round of processing and some other changes to the designs, to increase its resistance to collision attacks. 

Let's dive into MD5 algorithm design...
#### MD5
![[Pasted image 20250907094316.png]]
MD5 is designed to take an input of any length and produce a 128-bit fixed hash value. MD5 is identified as the Internet Standard RFC 1321.

This algorithm's workflow can be explained in 5 steps,
1. Padding the input message
As I mentioned earlier, MD5 takes inputs of any sizes. 
As the first step of the algorithm, the input message is padded so its length is congruent to 448 mod 512. 
(Notice that 448 is 64 less than 512. The reason will be explain in the next step.)

The padding process involve,
- Append a single "1" bit to the end of the message.
- Append enough "0" bits to make the length congruent to 448 mod 512. 

2. Append a 64-bit length value
The original message's length(in bits) is appended as a 64-bit value.
Since 448 + 64 = 512 bits, the final padded message will be a multiple of 512 bits. 

3. Initialize the MD buffer
MD5 uses four 32-bit buffers(total 128-bits).
These are called "chaining variables" or "Initialization values"(IVs).
Often labeled as A,B,C,D and initialized with specific hexadecimal values before processing any message data,
- A = `0x67452301`
- B = `0xEFCDAB89`
- C = `0x98BADCFE`
- D = `0x10325476`
These values are not random values, instead they are specifically chosen constants defined in the MD5 specification. 
(Many cryptographic algorithms uses constant to,
- Break symmetry between the registers
- Provide a good mixing at the starting point
- Avoid simple values like 0s or sequential values that might weaken the algorithm. )

3. Process the message in 512-bits block
At the end of the 2nd step, our message has became a multiple of 512 bits. 
- Now this padded message is split into 512-bit chunks. 
Ex: padded message -> M[0], M[1], M[2], ... , M[n]
- Then each 512-chucks are again split into 16 words of 32-bit blocks. 
Ex: M[i] -> W[0], W[1], W[2],...,W[15]

Then MD5 apply total 64 operations to each of these 512 block.
Above 64 operations grouped into 4 rounds 16 operations. 
Each operation group uses several words(W[i]) but in different permutations.

Ex: 
	Round 1 (16 operations) -> Uses W[0],...,W[15] in order.
	Round 2 (16 operations) -> Uses W[1], W[6], W[11],...In a different sequence.
	Round 3 (16 operations) -> Yet another ordering.
	Round 4 (16 operations) -> another ordering.

This process ensures every 32-bit word influences the final hash in multiple ways, not just once. 

**There are few crucial concept in this process we have to understand.**

Each round uses it's own nonlinear functions(F, G, H, I).
All 64 operations mixes up,
- one of the registers(A, B, C, D)
- One message word(W[i] from M[n])
- A constant(from since table)
- A left rotation

Basically, it's not correct to say one 32-bit word(W[i]) gets 4 operations so 16 x 4 = 64 total operations.
Instead, 
- Those 16 words are re-used across 64 operations.
- The re-use happens in different patterns across 4 round. 

During each of the above mentioned operations, registers(A,B,C, D) get updated with the help of above mentioned non-linear functions(F,G,H,I).

To explain this process more, 
Each operation is looks like this,
$$
A = B + ((A + F(B,C,D) + W[K] + T[i] <<< S))
$$
	A,B,C,D : Four 32-bit registers
	F,G,H,I : Four round functions
	W[K] : One of the 12 words from the current 512-bit message chunk
	T[i] : A constant for step 'i'(pre-computed from the sine function)
	S : Number of bits to left rotate(different for each step)
	<<<< S : Left circular shifting S times. 

![[Pasted image 20250907105830.png]]

After each operation, the order/place/role of the registers(A,B,C,D) gets rotated. Like,
	Operation 1 -> (A,B,C,D)
	Operation 2 -> (B,C,D,A)
	....
	Operation 3 -> (C,A,B,D)

This rotation makes sure, each register plays a different role in the next step and ensure all four registers get mixed. 

To get an understand about how these non-linear functions(F,G,H,I) are working with registers, look at the below example.
Ex:
Round 1(Operations 0 - 15)
	Function : F(B,C,D) = $(B  \quad \text{AND} \quad C) \quad \text{OR} \quad ((\text{NOT B}) \quad \text{AND} \quad D)$   
	Uses message words : W[0], ... W[15] in order
Round 2(Operations 16 - 31)
	Function : G(B,C,D) = $(B  \quad \text{AND} \quad D) \quad \text{OR} \quad (C \quad \text{AND} \quad  (\text{NOT D}))$   
	Uses message words : W[1], W[6], W[11],...
Round 3(Operations 32 - 47)
	Function : H(B,C,D) = $(B \quad \text{XOR} \quad C \quad \text{XOR} \quad D)$   
	Uses message words : W[5], W[8], W[11],... 
Round 4(Operations 48 - 63)
	Function : I(B,C,D) = $(C \quad \text{XOR} \quad (B \quad \text{OR} \quad \text{NOT D}))$   
	Uses message words : W[0], W[7], W[14],... 

5. Final Output
As I mentioned in earlier steps, 
A, B, C, D registers get changed while operations are executing the help of F,G,H,I non-linear functions. 
These processes are apply to all 512-word chunks. 
At then end of all operations, above mentioned A,B,C,D buffers get concatenated.
Since these were 32-bit buffers. Total concatenated hash will be 128-bits. 

*****
Ex: 
message: "Hello World"
ASCII bit code : 
![[Pasted image 20250907100848.png]]
This message contains 88 bits. 
It requires additional 360 bits to make it 448 bits.
Here how it looks after padding the message.
![[imageedit_1_9783435996.png]]
Notice the there's 1 x (10000000) and 44 x (00000000).
In another words, there's one 1 and 359 zeros. 

The original message was 88-bits.
88 in binary is `1011000`
Since we have to use 64 bits to represent the original message size,
Length part will be,
![[Pasted image 20250907123239.png]]

After appending the 64-bits long length to the 448-bit long message,
![[imageedit_2_9633921319.png]]

Then this total message will be divide into 512-bits chunks.
In this example, it will be only one 512-bit chunks. 
Then each chunk will be again divide into 16 of 32-bit words. 
![[Screenshot_2025-09-07_10-22-46.png]]

Then it will initialize the MD buffers,
- A = `0x67452301`
- B = `0xEFCDAB89`
- C = `0x98BADCFE`
- D = `0x10325476`

Then each message chunk will be processed as the step 4 mentioned and meanwhile, above MD buffers will change their values as a result. 
- A = `0xe59ff979`
- B = `0x41044f85`
- C = `0xdf5297e1`
- D = `0xc302d260`

Hash will be the concatenation of above MD buffers, 
Hash = `e59ff97941044f85df5297e1c302d260`


https://medium.com/@rangamudunkotuwa1729/md5-hashing-algorithm-721464ce9f73