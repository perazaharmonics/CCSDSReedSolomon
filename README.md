## Description
CCSDS GF(256) originally developed by Phil Karn (KA9Q) in C, then translated by Ryan Crosby to C#. 
This was later translated by myself to C++, and extended to include the RS(255,223) decoder.

## Mathematical Foundation

### Overview
Reed-Solomon codes are maximum-distance-separable (MDS) codes widely used in space communications, data storage, and error correction applications. This implementation provides $\text{RS}(255,223)$ encoding and decoding over the finite field $\mathbb{F}_{2^8}$, which is the CCSDS-standardized configuration for telemetry channel coding.

### Code Parameters
This implementation targets the CCSDS Reed-Solomon code $\text{RS}(255,223)$, which operates over the finite field $\mathbb{F}_{2^8}$:

| Parameter | Value | Description |
|-----------|-------|-------------|
| n | 255 | Codeword length (symbols) |
| k | 223 | Information/message length (symbols) |
| m | 8 | Field extension degree (bits per symbol) |
| 2t = n - k | 32 | Number of parity check symbols |
| t | 16 | Error-correction capability (symbol errors) |

The code achieves the Singleton bound: $d = n - k + 1 = 33$, meaning it can correct up to 16 symbol errors or detect up to 32 errors. Each symbol represents 8 bits, so actual bit-level error correction depends on error patterns.

### Finite Field $\mathbb{F}_{2^8}$ Arithmetic
All arithmetic is performed in the Galois field $\mathbb{F}_{2^8}$, where each element is represented as an 8-bit byte. The field is constructed via an irreducible polynomial (typically $x^8 + x^4 + x^3 + x^2 + 1$ in CCSDS). Multiplication and division are performed modulo this polynomial, with special lookup tables for efficiency.

Two representations are supported:
- **Conventional basis**: Standard polynomial basis representation used in many implementations
- **Dual basis**: Faster arithmetic operations on hardware, required by some CCSDS interfaces

### Encoding Process
The encoder accepts a message $m(x)$ of degree at most $k-1$ and produces a codeword $c(x)$ of degree at most $n-1$:

1. **Message polynomial**: $m(x) = m_0 + m_1 x + \cdots + m_{k-1}x^{k-1}$
2. **Generator polynomial**: $g(x) = \prod_{i=0}^{2t-1} (x - \alpha^{b+i})$, where $\alpha$ is a primitive root and $b$ is a starting offset
3. **Parity computation**: $r(x) = (x^{2t} \cdot m(x)) \bmod g(x)$
4. **Codeword**: $c(x) = x^{2t} \cdot m(x) + r(x)$

The result is a valid codeword divisible by $g(x)$.

### Decoding Process
The decoder receives a potentially corrupted word $r(x)$ and recovers the original message $m(x)$:

1. **Syndrome computation**: $S_i = r(\alpha^{b+i})$ for $i = 0, \ldots, 2t-1$
   - If all syndromes are zero, no error is detected
   - Otherwise, syndromes encode error location and magnitude information

2. **Key equation solving** (for $t > 0$ errors):
   - Berlekamp–Massey algorithm computes the error-locator polynomial $\Lambda(x)$
   - Chien search finds the roots of $\Lambda(x)$ (error positions)
   - Forney algorithm computes error magnitudes from syndromes and $\Lambda(x)$

3. **Error correction**: Subtract computed error magnitudes from the received word at error positions

4. **Message recovery**: Extract the first $k$ symbols of the corrected codeword

### Basis Conversion
CCSDS systems often interface between conventional and dual basis representations. This implementation explicitly handles conversion in both encoder and decoder paths for $\text{RS}(255,223)$, ensuring interoperability with CCSDS-compliant systems.

## CCSDS Reference

Primary normative source:

- CCSDS 131.0-B-5, TM Synchronization and Channel Coding (Blue Book), Consultative Committee for Space Data Systems.

This standard defines the channel coding framework used in space telemetry links, including Reed-Solomon coding parameters and related interoperability requirements.

### References
https://github.com/crozone/ReedSolomonCCSDS/tree/master
"General purpose Reed-Solomon decoder for 8-bit symbols or less", Copyright 2003 Phil Karn, KA9Q

## Credits and Reference
Thank you to all those that came before me, namely:

Phil Karn KA9Q

Ryan Crosby @crozone
https://github.com/crozone



