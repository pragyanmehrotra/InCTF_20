# DLPoly (676 pts)
RSA is easy. DLP is hard.
Challenge files: out.txt

## Approach

Looking at out.txt

```
sage: p
35201
sage: len(flag)
14
sage: X = int.from_bytes(  flag.strip(b'inctf{').strip(b'}') ,  'big')
sage: n
1629*x^256 + 25086*x^255 + 32366*x^254  + ... + 16286*x + 12050
sage: g
x
sage: g^X
10254*x^255 + 11436*x^254 + 9453*x^253 + ... + 549*x + 1468
```

Well it's clear from the out.txt file that we are looking for the integer X. This problem is also called as the [discrete logarithmic problem](https://en.wikipedia.org/wiki/Discrete_logarithm). Well looking at the given information we first deduce the upper bound for X, since we are hinted about it's generation. We see that, the resulting string would be of 7 characters and the max value it can attain is `"\xff"*7`	which is approximately of the order `10^{17}` which is pretty small.

Discrete logarithm has always been challenging incase of integers as well. So I had this idea initially that if we could factor n and compute the discrete logarithm with respect to each factor then I should be able to find my desired result, checkout [Pohlig-hellman algortihm](https://en.wikipedia.org/wiki/Pohlig%E2%80%93Hellman_algorithm), I had no idea about this while solving (would've made things a lot easier), I remembered reading a research for integers with known prime factors to compute the discrete log and was trying to draw an analogy from there.

Here's the solution in sage

```python
p = 35201
n = # checkout out.txt
c = # g^X from out.txt
P.<x> = PolynomialRing(GF(p))
nx = P(n)
px1 = list(nx.factor())[0][0]
###OUTPUT###
#x^6 + 6522*x^5 + 30239*x^4 + 20795*x^3 + 11104*x^2 + 8797*x + 1288
############
Q.<x> = QuotientRing(P, P.ideal(px1))
cx = Q(c)
#elements don't calculate there order themselves for QuotientRings so we have to guess the order, we know that it must divide (|Q| - 1)
#most likely the order is large so we will traverse in decreasing order so that we stumble upon it faster
for i in divisors(Q.order() - 1)[::-1]: 
	try:
		discrete_log(cx, x, ord = i)
		break
	except:
		continue
####OUTPUT####
#   108204963360545
##############

import binascii
print (binascii.unhexlify(hex(108204963360545)[2:]))
#>>> bingo!
#FLAG: inctf{bingo!}
```

`FLAG: inctf{bingo!}`