# PolyRSA (100 pts)
all that warmup you need to get started
challenge files: out.txt

## Approach
Looking at out.txt
```
sage: p
2470567871
sage: n
1932231392*x^255 + 1432733708*x^254 + ... + 1045363399*x + 1809685811
sage: m^65537
1208612545*x^254 + 1003144104*x^253 + ... + 776654074*x + 886398299 
```
The challenge title and the data given makes it clear that we have to solve the standard [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) problem but in the field of polynomials. I believe if some of you are interested you may find [this paper](http://www.diva-portal.se/smash/get/diva2:823505/FULLTEXT01.pdf) useful. I will be covering only the topics aiding in solving the challenge.

Now, we could clearly see that the given polynomials for $n$ and $c = m^{65537}$ all have there coefficients smaller than the given $p$ this is because the Polynomial ring has been defined in $\mathbb{Z}_p$. Also you may observe that that degree of $c$ is not more than 254 this is because the further operations are happening in the quotient ring with ideal as $n$. 

Basically, in case of Integers, we can represent any number $a = qn + r$ for any $n$ and a unique pair of $(q,r)$ thus we can then represent $a \equiv r \bmod n$ Since on dividing $a$ with $n$ we get a remainder $r$. Similarly, we can say in the case of polynomials, that every polynomial $f(x) = q(x)n(x) + r(x)$
thus, $f(x) \equiv r(x) \bmod n(x)$

Now, moving on to the solution, as in the case of standard RSA, polynomial RSA also reduces to factorizing the polynomial, Let's say we are able to factor $n(x) = q(x)h(x)$ where $q(x), h(x)$ are irreducible polynomials. As in case of standard RSA we have $n=pq$. We know that, $a^{\phi(n)} \equiv 1 \bmod n$ for any $a$ having $gcd(a,n) = 1$ where $\phi(n) = (p-1)*(q-1)$ or the Euler's phi function, denoting the number of invertible integers. Similarly in case of polynomials we find the number of invertible polynomials. For the polynomial $f(x)$ to be invertible in the quotient ring of $n(x)$ the $gcd(f(x), n(x)) = 1$ and there must exist another polynomial g(x) such that $f(x)g(x) \equiv 1 \bmod n$. 

The paper I recommended earlier covers this topic in depth and proves how to find such an inverse. For an irreducible polynomial $q(x)$ in $\mathbb{Z}_p$ the number of invertible polynomials are given by $(p^{deg(q(x))} - 1)$ this is because all polynomials smaller than $q(x)$ are invertible except 0 and the coefficient can't be greater than or equal to $p$ as we are in the field $\mathbb{Z}_p$. Thus, for our problem we get our new $\phi(n(x)) = (p^{deg(q(x))} -1)*(p^{deg(h(x))} -1)$. Finally, we compute $d$ such that $ed \equiv 1 \bmod \phi(n(x))$ and thus we retrieve our message $m(x)$ as, $c(x)^d \equiv m(x) \bmod n(x)$.

Below, is the implementation of the above idea in sage.
```python
p = 2470567871
P.<x> = PolynomialRing(GF(p))
n = #the polynomial n(x) look at out.txt
c = #the polynomial m^65537 from out.txt
e = 65537
nx = P(n)
factor(nx)
###output####
#(1932231392) * (x^127 + 483092954*x^126 + 783188479*x^125 + ... + 663555668) * (x^128 + 627460182*x^127 + 1594545998*x^126 + ... + 951119341*x + 2464448261)
#############
#We only care about the degrees of the factors (we can ignore the constant factor)
phi = (pow(p, 128) -1)*(pow(p, 127) - 1)
d = inverse_mod(e, phi)
Q.<x> = QuotientRing(P, P.ideal(nx))
cx = Q(c)
m = pow(cx, d)
flag = ""
for i in m.coefficients():
	flag += chr(i)
print ("FLAG:",flag)
```

`FLAG: inctf{and_i_4m_ir0n_m4n}`