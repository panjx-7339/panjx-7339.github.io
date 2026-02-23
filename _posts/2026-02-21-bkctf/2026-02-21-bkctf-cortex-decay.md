---
title: "bkctf — RSA with Weak Primes Writeup"
date: 2024-02-22
categories: [CTF, bkctf]
tags: [crypto, rsa, factorization, python, sympy]
description: Breaking RSA by factoring a modulus composed of 8 weak prime factors.
math: false
mermaid: false
pin: false
---

**Challenge name:** Cortex Decay

**Category:** Crypto

## Challenge Description

We're given a standard RSA setup with a ciphertext, public exponent, and modulus. The hint reads:

> "Some people question my choice of primes, but I know I'm right."

**Given values:**

```
n = 67000000000000000000000000245061662851489575612371642903203727663237160203426
e = 65537
ciphertext = 40732687938760268194816992508783308058844901710443215136378413744389173154801
```

## Initial Analysis

The hint is wordplay — "I know I'm **right**" suggests **right-truncatable primes**: a special class of primes where the number remains prime as digits are successively removed from the right. Only 83 such primes exist, making them a bad choice for RSA.

More importantly, a well-formed RSA modulus should be the product of exactly **two large, random primes**. Any deviation — small factors, too many factors, patterned primes — makes the modulus trivially factorable.

The first red flag is the structure of `n` itself: it begins with `67` followed by a long run of zeroes before a smaller number, hinting at a non-random construction.

## Factoring the Modulus

Using `sympy.factorint`, we attempt to factor `n`:

```python
import sympy

n = 67000000000000000000000000245061662851489575612371642903203727663237160203426
factor_dict = sympy.factorint(n) # contains factors of n with their respective multiplicities
print(factor_dict)
```

This returns:
```
{2: 1, 3: 1, 67: 1, 1483: 1, 14180303: 1, 40938258341: 1, 
1324437742957822811: 1, 146170986161787706448601731202221987: 1}
```

Thus, the **complete factorization** of `n` is:

```
n = 2
  × 3
  × 67
  × 1483
  × 14180303
  × 40938258341
  × 1324437742957822811
  × 146170986161787706448601731202221987
```

Eight prime factors instead of two — RSA is completely broken.

## Computing the Private Key

With all factors known, we compute Euler's totient φ(n) and the private exponent `d`:

```python
factors = list(factor_dict.keys())

phi = 1
for p in factors:
    phi *= (p - 1)

e = 65537
d = pow(e, -1, phi)
```

## Decryption

Standard RSA decryption: `m = ciphertext^d mod n`, then convert the integer to bytes:

```python
ct = 40732687938760268194816992508783308058844901710443215136378413744389173154801
m = pow(ct, d, n)

h = hex(m)[2:]
if len(h) % 2:
    h = '0' + h
flag = bytes.fromhex(h).decode()
print(flag)
```

## Full Solve Script

```python
import sympy

ct = 40732687938760268194816992508783308058844901710443215136378413744389173154801
e = 65537
n = 67000000000000000000000000245061662851489575612371642903203727663237160203426

factor_dict = sympy.factorint(n)
print(factor_dict)

# compute euler's function and private exponent
phi = 1
for p in list(factor_dict.keys()):
    phi *= (p-1)

d = pow(e, -1, phi)
m = pow(ct, d, n)

h = hex(m)[2:]
if len(h) % 2:
    h = '0' + h
flag = bytes.fromhex(h).decode()
print(flag)

```

## Flag

`bkctf{6ig-numbers-are-6e77er}`

The flag itself is the punchline — "big numbers are better" — a reminder that RSA security depends entirely on using large, randomly chosen primes, not small or patterned ones.

## Key Takeaways

- RSA requires exactly **two large, random primes**. Using special primes (right-truncatable, sequential, patterned) defeats the entire scheme.
- A modulus with many small factors is trivially broken with `sympy.factorint`.
- CTF hints are often wordplay. "I know I'm right" → right-truncatable primes.