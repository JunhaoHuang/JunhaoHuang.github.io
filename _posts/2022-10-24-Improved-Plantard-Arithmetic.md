---
layout: post
title: Integrating the Improved Plantard Arithmetic into Kyber
subtitle: Some steps required during the integration 
gh-repo: daattali/beautiful-jekyll
gh-badge: [star, fork, follow]
tags: [Kyber, PQC, Modular arithmetic]
comments: true
---

## Background
Modular arithmetic is a key fundamental operation in cryptography. The state-of-the-art modular arithmetic is Montgomery[1] and Barrett arithmetic[2]. They both use the relatively cheaper shift operation to replace the expensive division operations in the traditional modular arithmetic, which helps to speed up the modular arithmetic. In this article, we focus on the word-size moduli that are smaller that $2^{l},l=16,32,64$. The state-of-the-art Montgomery and Barrett arithmetic for word-size moduli are shown below.

---
>**Input:** $a,b$ such that $-2^{l-1}q\leq ab <2^{l-1}q$, where $l$ is the machine word size $l=16,32$ or $64$, the odd modulus $q\in (0,2^{l-1})$.  
**Output:** $r\equiv ab2^{-l}\bmod q, r\in (-q,q)$.  
mont_mul($a,b$):  
$c=c_1 2^l+c_0=ab$  
$m=c_0 q^{-1} \bmod^{\pm} 2^l$  
$t_1=\lfloor mq/2^l\rfloor$  
$r=c_1-t_1$  
$\text{return } r$


---
>**Input:** $a,b$ such that $0\leq ab <2^{2l^{\prime}+\gamma}$, where $l^{\prime}$ and $q$ satisfy $2^{l^{\prime}-1}<q<2^{l^{\prime}}$, the precomputed constant $\lambda= \lfloor 2^{2l^{\prime}+\gamma}/q\rfloor$.  
**Output:** $r\equiv ab\bmod q, r\in [0,q]$.  
Barrett_mul($a,b$):  
$c=ab$  
$t=\lfloor c\lambda/2^{2l^{\prime}+\gamma}\rfloor$  
$r=c_1-t_1$  
$\text{return } r$


The state-of-the-art Montgomery and Barrett multiplication both require 3 multiplications, 1 addition/subtraction and some cheap shift or modulo operations. The Barrett multiplication can be modified to support signed integers as shown in [3]. The reason why they are popular in cryptography is that they supports signed integers in a large domain, which can be used to speed up the butterfly units using signed integers in the Lattice-based Cryptography (LBC). 

Recently, Thomas Plantard [4] proposed a new word-size modular arithmetic (i.e., Plantard arithmetic) that can save one multiplication operation when multiplying a constant. It is suggested that we can speed up operations that require the frequent usage of the modular multiplication by a constant, e.g., the modular multiplication by the twiddle factors in LBC. The original Plantard arithmetic is shown as follow.

---
>**Input:** $a,b\in [0,q], q<\frac{2^l}{\phi}, q^{\prime}\equiv q^{-1} \bmod 2^{2l}$, where $l$ is 16, 32 or 64.  
**Output:** $r\equiv ab(-2^{-2l}) \bmod q, r\in [0,q)$.  
plant_mul($a,b$):  
$r=[([[abq^{\prime}]_{2l}]^l+1)q]$ // $[x]_l\equiv x \bmod 2^l, [x]^l\equiv x>>l$.  
if $r=q$:  
$\quad \text{return } 0$  
$\text{return } r$


We can see that the origin Plantard multiplication for two variables takes 3 multiplication, 1 addition, 1 modulo by $2^{2l}$ and 1 shift operation. The major difference between Plantard arithmetic and Montgomery or Barrett arithmetic is that the product $ab$ is only used once. So, when $b$ is a constant, then $bq^{\prime}$ can be precomputed mod $2^{2l}$ and we can save one multiplication operation by precomputing $bq^{\prime}$. However, the original Plantard arithmetic only supports unsigned integers. It would require extra addition by a multiple of $q$ during the butterfly units to ensure the the subtraction would not produce negative results. Besides, the original Plantard multiplication only supports inputs in a small domain $[0,q]$, which requires expensive modular reduction after each layer of butterfly unit. Therefore, the original Plantard arithmetic is impractical in LBC. 

We have improved the Plantard arithmetic to support larger input domain in signed integers and the related paper has been published in [TCHES 2022](https://ches.iacr.org/2022/) [5]. The improved Plantard arithmetic is shown below.

---
>**Input:** $a,b\in [-q2^{\alpha},q2^{\alpha}], q<2^{l-\alpha-1}, q^{\prime}\equiv q^{-1} \bmod^{\pm} 2^{2l}$, where $l$ is 16, 32 or 64.  
**Output:** $r\equiv ab(-2^{-2l}) \bmod^{\pm} q, r\in (-\frac{q}{2},\frac{q}{2})$.  
improved_plant_mul($a,b$):  
$r=[([[abq^{\prime}]_{2l}]^l+2^{\alpha})q]$ // $[x]_l\equiv x \bmod 2^l, [x]^l\equiv x>>l$.  
$\text{return } r$


In this algorithm, we first put a stricter constraint over $q$ such that $q<2^{l-\alpha-1}$. Then, we also modify the correctness proof based on the new modulus restriction and enlarge the input range up to $[-q2^{\alpha},q2^{\alpha}]$. As for the main steps, we only need to modify the $+1$ to $+2^{2^\alpha}$, which will not affect the total complexity. Besides, the output of this algorithm is reduced down to $(-\frac{q}{2},\frac{q}{2})$, which eliminate the final correction step in the original version. With these tweaks over the Plantard arithmetic, we are able to integrate the Plantard arithmetic efficiently into LBC, which may lead to even better LBC implementation compared to the implementation with the state-of-the-art Montgomery or Barrett arithmetic on some platforms.

## Integration to Kyber on Cortex-M4
For details of how we can efficiently integrate the improved Plantard arithmetc into the 16-bit NTT in Kyber on the Cortex-M4 platform, please check our paper [5](/assets/paper/TCHES2022.pdf).
### Cortex-M4

## References
[1] Montgomery P L. Modular multiplication without trial division[J]. Mathematics of computation, 1985, 44(170): 519-521.  
[2] Barrett P. Implementing the Rivest Shamir and Adleman public key encryption algorithm on a standard digital signal processor[C]//Conference on the Theory and Application of Cryptographic Techniques. Springer, Berlin, Heidelberg, 1986: 311-323.  
[3] <https://github.com/pq-crystals/kyber/blob/master/ref/reduce.c>  
[4] Plantard T. Efficient word size modular arithmetic[J]. IEEE Transactions on Emerging Topics in Computing, 2021, 9(3): 1506-1518.  
[5] Huang J, Zhang J, Zhao H, et al. Improved Plantard Arithmetic for Lattice-based Cryptography[J]. IACR Transactions on Cryptographic Hardware and Embedded Systems, 2022, 2022(4): 614-636.  
