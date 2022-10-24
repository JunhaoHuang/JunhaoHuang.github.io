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
Thomas Plantard proposed a new word-size modular arithmetic (i.e., Plantard arithmetic) that can save one multiplication operation when multiplying a constant. It is suggested that we can speed up operations that require the frequent usage of the modular multiplication by a constant. The original Plantard arithmetic is shown as follow.

```
// $a,b\in [0,q], q<\frac{2^l}{\phi}, q^{\prime}\equiv q^{-1} \bmod 2^{2l}$, where $l$ is 16, 32 or 64.
plant_mul($a,b$):
	$r=[([[abq^{\prime}]_{2l}]^l+1)q]$
	if $r=q$:
		return 0
	return $r\equiv ab(-2^{-2n}) mod q$ where $r\in[0,q)$
```

When $b$ is a constant, then $bq^{\prime}$ can be precomputed mod $2^{2l}$.

I have been working 
