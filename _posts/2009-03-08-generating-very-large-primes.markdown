---
author: admin
comments: true
date: 2009-03-08 01:02:09+00:00
layout: post
slug: generating-very-large-primes
title: Generating (Very) Large Primes
wordpress\_id: 167
tags:
- crypto
- python
- ssl
---

Have you ever wondered how big the "large primes" that RSA encryption is based on really are?  What exactly does a "1024-bit" key mean anyway?  And if the difficulty of RSA is partially based on [factoring large numbers](http://en.wikipedia.org/wiki/Integer_factorization), how do we create these large primes without determining primality via factorization?

The easiest way to demonstrate these concepts is with a simple script, so let's take a look at a large random number generator I wrote[^1] using Python.  As is typical for this blog, you'll note that the random numbers are not cryptographically secure.  To make these examples simple I don't want to introduce external dependencies, but please keep that issue in mind for any serious applications of the code presented here.

```python
import random
import math
import sys

def rabinMiller(n):
     s = n-1
     t = 0
     while s&1 == 0:
         s = s/2
         t +=1
     k = 0
     while k<128:
         a = random.randrange(2,n-1)
         #a^s is computationally infeasible.  we need a more intelligent approach
         #v = (a**s)%n
         #python's core math module can do modular exponentiation
         v = pow(a,s,n) #where values are (num,exp,mod)
         if v != 1:
             i=0
             while v != (n-1):
                 if i == t-1:
                     return False
                 else:
                     i = i+1
                     v = (v**2)%n
         k+=2
     return True

def isPrime(n):
     #lowPrimes is all primes (sans 2, which is covered by the bitwise and operator)
     #under 1000. taking n modulo each lowPrime allows us to remove a huge chunk
     #of composite numbers from our potential pool without resorting to Rabin-Miller
     lowPrimes =   [3,5,7,11,13,17,19,23,29,31,37,41,43,47,53,59,61,67,71,73,79,83,89,97
                   ,101,103,107,109,113,127,131,137,139,149,151,157,163,167,173,179
                   ,181,191,193,197,199,211,223,227,229,233,239,241,251,257,263,269
                   ,271,277,281,283,293,307,311,313,317,331,337,347,349,353,359,367
                   ,373,379,383,389,397,401,409,419,421,431,433,439,443,449,457,461
                   ,463,467,479,487,491,499,503,509,521,523,541,547,557,563,569,571
                   ,577,587,593,599,601,607,613,617,619,631,641,643,647,653,659,661
                   ,673,677,683,691,701,709,719,727,733,739,743,751,757,761,769,773
                   ,787,797,809,811,821,823,827,829,839,853,857,859,863,877,881,883
                   ,887,907,911,919,929,937,941,947,953,967,971,977,983,991,997]
     if (n >= 3):
         if (n&1 != 0):
             for p in lowPrimes:
                 if (n == p):
                    return True
                 if (n % p == 0):
                     return False
             return rabinMiller(n)
     return False

def generateLargePrime(k):
     #k is the desired bit length
     r=100*(math.log(k,2)+1) #number of attempts max
     r_ = r
     while r>0:
        #randrange is mersenne twister and is completely deterministic
        #unusable for serious crypto purposes
         n = random.randrange(2**(k-1),2**(k))
         r-=1
         if isPrime(n) == True:
             return n
     return "Failure after "+`r_` + " tries."

print generateLargePrime(1024)
```

This code is very slow, but does not represent an entirely naïve approach.[^2]  To generate a prime we first create a random integer in the range (2k-1,2k), then the following rules are applied:





  1. The number (n) must be >=3.  While 2 is a prime number, for our purposes we have no interest in numbers less than 3.


  2. Do a bitwise and (n&1).  If the result is not 0 then we know the number is even and can throw it out.


  3. Check that n%p is 0 (in other words, that n is not divisible evenly by p) for all primes <1000.  This check will eliminate a large quantity of composite numbers and prevent the expense of the next test.


  4. Finally we reach the core test: [Rabin-Miller](http://en.wikipedia.org/wiki/Miller–Rabin_primality_test).  This algorithm, if it returns true, states that there is a 75% chance that the number is prime (for the randomly chosen basis a).  To obtain a 2-128 chance that the number is not prime, we must repeatedly run the Rabin-Miller test choosing different number bases.  Since each iteration of the test increases our probability by a power of 2 we must iterate 64 times to reach this confidence level.


 
If the number passes all these tests it is returned as a valid prime number.  Of course, if you don't trust the output from this script you can always check it with openssl...

```bash
openssl prime 7337488745629403488410174275830423641502142554560856136484326749638755396267050319392266204256751706077766067020335998122952792559058552724477442839630133
8C18E5DC98684E2A15B84535635A95C4A192B73B40A780AB4CB0C58BDB9C31EF970C3AC6D804712B830FB6F1B140693A251E989F89B687EBA62781AD031D5135 is prime
```


[Click](/assets/media/2009/03/8192_prime.txt) for an example of an 8192-bit prime created with the generateLargePrime() function.  You can also check out a [1024-bit prime](/assets/media/2009/03/1024_prime.txt) as well.  1024-bit keys are the minimum size recommended for end entity certificates using RSA (SSL certificates) at this time.[^3]

Of course, a large prime is nice, but it isn't necessarily an RSA prime.  Look for another entry soon explaining the additional restrictions imposed by RSA.

[^1]: This code is based on the pseudocode provided in [Practical Cryptography](http://www.amazon.com/Practical-Cryptography-Niels-Ferguson/dp/0471223573/ref=pd_bbs_sr_4?ie=UTF8&s=books&qid=1236471581&sr=8-4).

[^2]: Generation of a 1024-bit prime using this script takes 5-10 seconds, whereas OpenSSL takes 0.1-0.2 seconds.

[^3]: NIST guidelines suggest migration to 2048-bit keys by 31 Dec 2010)
