While studying the physical layer of the networking I came across with this equation to find the maximum data rate of a noiseless channel.
$$
\text{Maximum Data rate of a noiseless channel} = 2 \cdot B \cdot \log_{2}{M}
$$
M : Number of discrete levels in the signal
B : Bandwidth of the channel


This made me wonder:
- How was this equation derived?
- Why is $B \cdot log⁡_{2}M$ multiplied by 2?
- What is bandwidth?
- What are discrete signal levels?
While researching these questions, I encountered the concept of signal sampling. Understanding signal sampling helped me make sense of the equation above.

## Key Concepts Before Understanding Signal Sampling
#### 1. Bandwidth
The range of frequencies that a communication channel can transmit. 
Bandwidth represent the difference between the highest and lowest frequencies the channel can carry. 
Ex: If a channel can transmit between 300Hz to 3300Hz, 
Bandwidth = 3300Hz - 300Hz = 3000Hz
- Frequency -> Number of signal oscillates per second
- High frequency -> More oscillates per second
High bandwidth shows a high frequency.
$\therefore$ High bandwidth -> More signal oscillates per second
$\therefore$ High bandwidth -> More data per second(high data rate)
#### 2. Discrete signal levels
Refers to the number of distinct voltage or signal values used in digital transmission. 
Instead of using just two levels(0,1) we can use multiple levels to encode mote bits per signal. 
![[Pasted image 20250403130150.png]]
- This diagram has only two amplitude levels.(level 1 and level 2)
- Each level represents a single bit(0 for level 1 and 1 for level 1)
- Since there are 8 transitions in 1 second, 8 bits are sent in 1 second.
- This results in a bit rate of 8 bps. 
![[Pasted image 20250403130522.png]]
- This diagram shows four amplitude levels(Level 1 to Level 4)
- 4 Levels can be represented by 2 bits. ($\log_{2}{4} = 2$)
	Level 1 = `00`
	Level 2 = `01`
	Level 3 = `10`
	Level 4 = `11`
- Since there are 8 transitions in 1 second, and each transition now carries 2 bits, the total bit count is $2 \times 8 = 16$

Image Resource : https://mauc.edu.iq/wp-content/uploads/%D9%85%D8%AD%D8%A7%D8%B6%D8%B1%D8%A7%D8%AA%20%D9%87%D9%86%D8%AF%D8%B3%D8%A9%20%D8%A7%D9%84%D8%AD%D8%A7%D8%B3%D9%88%D8%A8/comp.net/comp.net4.pdf

More levels allow more bits to be send per signal, Increasing the data rate without increasing the bandwidth.
(But using too much levels can lead to signal distortion and errors, specially in noisy channels.)

Keeping these above concepts in my mind, we can move on to the Signal sampling. 
### Nyquist and Shannon
The maximum data rate of a noiseless channel can be calculated using the "Nyquist Theorem" about digital communication in 1928.
The foundation of Nyquist Theorem is the "Nyquist Sampling Theorem" about signal processing & sampling. It was later refined by Shannon Claude in 1949, identified as "Nyquist-Shannon Theorem" for band-limited signals. 

### What is signal sampling ?
Sampling is the process of converting a continuous-time analog signal into a discrete-time digital signal by taking measurement(samples/snapshots) at regular intervals. 

Ex: Source : https://www.youtube.com/watch?v=Jv5FU8oUWEY
![[Pasted Image 20250403145257_357.png]]
![[Pasted image 20250403150253.png]]
Sampling Frequency : Number of snapshots(samples) taken within a 1 second.
Signal Frequency : Number of oscillation per second(1Hz in this above example)
![[Pasted image 20250403150442.png]]
These each sample point is stored as a number, allowing digital devices to process and transmit. 

Now, if we try to plot 2Hz signal on top of the sampling points of 1Hz signal, it looks like this,
![[Pasted image 20250403151618.png]]
2Hz signal doesn't contain all these sampling points of 1Hz. 

If we increase the frequency to 3Hz, 4Hz, 5Hz, all of them doesn't line up with the sampling points 1Hz. 
But 6Hz does. 
![[Pasted image 20250403152239.png]]
Same alignment happen in 8Hz, 13Hz, 15Hz... etc. 
We call them **sinusoids**. 

Due to this reason, we can't determine what's the original wave only considering the sampling points. 
But if we analyze the list of sinusoids,
![[Pasted image 20250403152852.png]]
The original wave(1Hz) is the lowest in the list.

Now, let's try 4 Hz sampling rate without changing the original wave frequency. 
![[Pasted image 20250403183549.png]]

List of sinusoids align with these sample points,
$y = \cos{(2{\pi} \times [1] \ x)} \to 1 \ Hz$
$y = \cos{(2{\pi} \times [3] \ x)} \to 3 \ Hz$
$y = \cos{(2{\pi} \times [5] \ x)} \to 5 \ Hz$
$y = \cos{(2{\pi} \times [7] \ x)} \to 7 \ Hz$

In this scenario as well, the original wave frequency is the lowest frequency in the list.
![[Pasted image 20250403184615.png]]

But what if we decrease the sampling rate below 2 Hz (ex: 1.5 Hz),
![[Pasted image 20250403185235.png]]
Here, the original wave frequency is still 1 Hz.
Let's find out the sinusoids align with these sampling points,
$y = \cos{(2{\pi} \times [0.5] \ x)} \to 0.5 \ Hz$
$y = \cos{(2{\pi} \times [1] \ x)} \to 1 \ Hz$
$y = \cos{(2{\pi} \times [2] \ x)} \to 2 \ Hz$
$y = \cos{(2{\pi} \times [2.5] \ x)} \to 2.5 \ Hz$

![[Pasted image 20250403190754.png]]
In previous examples, lowest frequency was the original wave. But here it's not valid anymore.

When we sample signals at a rate higher than 2 Hz, we can observe that the lowest sinusoid corresponds to the original wave. However, when the sampling rate drops below 2 Hz, this phenomenon is no longer observable.

If we generalize the phenomena for any frequency(not only for 1Hz waves), we can say,
As long as our sampling rate is twice the frequency of the original wave, the sinusoid with the lowest frequency is the original wave between other sinusoids. 

#### Nyquist-Shannon sampling theorem :
To accurately reconstruct a signal, the sampling rate $(f_{s})$ must be at least twice the highest frequency $(f_{max})$ in the signal.
$$
Sampling \ rate \geq 2 \times f_{max}
$$
$$
f_{s} \ \geq \ 2 \ \times \ f_{max}
$$
This is the fundamental principle of digital signal processing to prevent aliasing(distortion caused when a signal is under-sampled).

In communication systems, signals are often band-limited. 
Instead of indexing from 0Hz to $f_{max}$ they typically exist within a frequency range such as,
$$
f_{min} \leq f \leq f_{max}
$$
$$
\therefore B \ (\ \text{bandwidth} \ ) = f_{max} \ - \ f_{min}
$$
If the signal start at 0Hz(base-band signal) then,
$$
B = f_{max}
$$
Substituting above result to the Nyquist sampling theorem,
$$
f_{s} = 2 \times B
$$
$$
\therefore Sampling \ rate \ge 2 \times B
$$
#### Considering signal levels and sampling together,
Each transmitter signal can represent one of $M$ discrete levels. 
Number of distinct levels  = $\log_{2}{M}$

To accurately re-construct a signal requires at least 2B signal sampling rate and, each signal change carries $\log_{2}{M}$ bits.
$$
\therefore \text{Maximum Data rate of a noiseless channel} = 2 . B .\log_{2}{M} 
$$

This is how we derive the formula for the maximum data rate of a noiseless channel using Nyquist-Shannon Sampling Theorem.

Example calculation :
If a 5Khz channel is used with 4 distinct levels.
$$M = 4$$
$$R_{max} = 2 \times (Bit \ rate) \ \times \ \log_{2}{M}$$
$$R_{max} = 2 \times 5000 \times \log_{2}{4}$$
$$R_{max} = 20,000 \ bps = 20Kbps$$
