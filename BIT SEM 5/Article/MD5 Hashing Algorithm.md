These days we are talking a lot about the possibility of quantum computers breaking current algorithms we use in day-to-day technology applications. While thinking about this new upcoming future, I heard about a classic example of how technology was improved back in 90s.
It was the story of Ronald Rivest and his MD hashing algorithm series. 

Let's dive in...
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
This new design, MD4 was simplified with some trade-offs for performance. 
This same reason, lead it's to have some severe security vulnerabilities. 
Within a year, cryptographers found weaknesses that could be exploited to find "collisions"(two different inputs that produce the same hash output) much faster than brute force. 

MD5(1991)
MD5 was developed as a direct response to the cryptographic vulnerabilities found in MD4 algorithm. The main focus of the new MD5 design was to retain the speed of MD4, while addressing it's security flaws. 