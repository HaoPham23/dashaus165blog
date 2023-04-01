# Cyber Apocalypse 2023: Elliptic Labyrinth Revenge

A writeup on Elliptic Labyrinth Revenge
<!--more-->


{{< admonition note "Challenge Information" >}}
* **Given materials:** [Get it here!](https://drive.google.com/file/d/1wKzblzA6_mYWHLM-CHcUo-6NjIzX9Llc/view?usp=sharing)
* **Description:** 
* **Category:** Crypto - Hard
{{< /admonition >}}

This challenge is a modified version of `Elliptic Labyrinth` to force CTF players solve it in intended way. 

The server script is shown below, which has a bit different from the previous version:
```python
import os, json
from hashlib import sha256
from random import randint
from Crypto.Util.number import getPrime, long_to_bytes
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from sage.all_cmdline import *
from secret import FLAG


class ECC:

    def __init__(self, bits):
        self.p = getPrime(bits)
        self.a = randint(1, self.p)
        self.b = randint(1, self.p)

    def gen_random_point(self):
        return EllipticCurve(GF(self.p), [self.a, self.b]).random_point()


def menu():
    print("1. Get parameters of path")
    print("2. Try to exit the labyrinth")
    option = input("> ")
    return option


def main():
    ec = ECC(512)

    A = ec.gen_random_point()
    print("This is the point you calculated before:")
    print(json.dumps({'x': hex(A[0]), 'y': hex(A[1])}))

    while True:
        choice = menu()
        if choice == '1':
            r = randint(ec.p.bit_length() // 3, 2 * ec.p.bit_length() // 3)
            print(
                json.dumps({
                    'p': hex(ec.p),
                    'a': hex(ec.a >> r),
                    'b': hex(ec.b >> r)
                }))
        elif choice == '2':
            iv = os.urandom(16)
            key = sha256(long_to_bytes(pow(ec.a, ec.b, ec.p))).digest()[:16]
            cipher = AES.new(key, AES.MODE_CBC, iv)
            flag = pad(FLAG, 16)
            print(
                json.dumps({
                    'iv': iv.hex(),
                    'enc': cipher.encrypt(flag).hex()
                }))
        else:
            print('Bye.')
            exit()


if __name__ == '__main__':
    main()
```

## Problem statement
The program generates random secret elliptic curve parameters and allows the user to:

  - Option 1: Obtain the modulus `p` and a few MSB bits of ECC parameters.

  - Option 2: Provide the encrypted `FLAG`.

Unlike the previous one, now the server doesn't provide an option to generate random points, instead it gives players only one point at the beginning. The objective is to recover curve's parameters given a single point of the curve, `p` and the most significant bits of `a` and `b`.
## Initial analysis

### AES Encryption
Easily see that the AES scheme is normal and therefore we can exploit anything from it. The only way to retrieve the `FLAG` is finding the key, which means finding the curve's parameters.
### Leak bits
For some $170 \leq r \leq 340$, let's define $a_{h}$ and $b_h$ as the $r$ MSB bits of $a$ and $b$, define $a_l$ and $b_l$ as the remaining bits of $a$ and $b$, respectively. By our definition, we have:

$a = a_h \times 2^r + a_l$

$b = b_h \times 2^r + b_l$

Substitute $a$ and $b$ to the Weierstrass elliptic curve equation, we get:

$y^2 \equiv x^3 + (2^ra_h + a_l)x + 2^rb_h + b_l \text{  (mod }p)$

We define a polynomial $F(\alpha, \beta)$ in $GF(p)$ satifies $F(a_l, b_l) = 0$:

$F(\alpha, \beta) = x_P\alpha + \beta + 2^r a_h \times x_P + 2^rb_h\times x_P - y^2_P$

where $(x_P, y_P)$ is the known point given by the server at beginning. When having most significant bits of a number known, a typical method to apply is Coppersmith, particularly bivariate polynomial Coppersmith in this time.
## Implementation and results
By connecting to the server, I received these information: 
```python
p = 0xe3b0aa3465a71f45fdd6350587d041c481ae061401465aa9e089827ac0548728771f6baf095b5f44bb8410dc9709ea22df72bf635f04475fedeb24f13d488ceb
a_h = 0x3128114d5bdecf9388699fd05d1432d444f9e8bda4e620b13445d6f9705721d7dff1
b_h = 0x2cf39f8fd105112fdaa7c7144f3e7e7da15e93fa59efc32b2c185bf5151153e7fd07
x_P = 0x266dd3ba72bad801e16d03509ae1656b0f137c2382f40a420ff90e40f291073b46ae395f2858ccd719299d786c8191796f882daf2a55760d9c58fbcb6c5355da
y_P = 0x7c24c560a9bf720ff447de5671342787c762508e44a2e269ed0794e5ef33f9014f1dd53d8a3ebcb301d5fecdfde4d2413ee079b0ad8e716729c0123787d7fa4d

iv = "8ace74afe026aab8ff1288a9076141fb"
enc = "a07c4ac6d8dc0abe11d955a79e37d8b21721704dfccf6f3938646c74b1c3374f6d0f8e71962e48c405c629533c804ea0"
```
The good implementation of multivariate Coppersmith I used is in [this repo](https://github.com/defund/coppersmith) of Defund:
```python
import itertools

def small_roots(f, bounds, m=1, d=None):
	if not d:
		d = f.degree()

	R = f.base_ring()
	N = R.cardinality()
	
	f /= f.coefficients().pop(0)
	f = f.change_ring(ZZ)

	G = Sequence([], f.parent())
	for i in range(m+1):
		base = N^(m-i) * f^i
		for shifts in itertools.product(range(d), repeat=f.nvariables()):
			g = base * prod(map(power, f.variables(), shifts))
			G.append(g)

	B, monomials = G.coefficient_matrix()
	monomials = vector(monomials)

	factors = [monomial(*bounds) for monomial in monomials]
	for i, factor in enumerate(factors):
		B.rescale_col(i, factor)

	B = B.dense_matrix().LLL()

	B = B.change_ring(QQ)
	for i, factor in enumerate(factors):
		B.rescale_col(i, 1/factor)

	H = Sequence([], f.parent().change_ring(QQ))
	for h in filter(None, B*monomials):
		H.append(h)
		I = H.ideal()
		if I.dimension() == -1:
			H.pop()
		elif I.dimension() == 0:
			roots = []
			for root in I.variety(ring=ZZ):
				root = tuple(R(root[var]) for var in f.variables())
				roots.append(root)
			return roots
	return []

r = len(bin(p)) - len(bin(a_h))
Fp = GF(p)
a_h, b_h, x_P, y_P  = map(Fp, [a_h, b_h, x_P, y_P])
P.<alpha, beta> = PolynomialRing(Fp)

F = x_P * alpha + beta + x_P^3 + 2^r * a_h * x_P + 2^r * b_h - y_P^2
roots = small_roots(F, (2^r, 2^r), m=2, d=5)[0]
a_l, b_l = roots
print(f'a_l = {a_l}')
print(f'b_l = {b_l}')
```
The results:
```python
a_l = 4090003137759760265604501674930222345811449862978588668280246527938919495
b_l = 6854882327443898686047723082547152279783184053818145063785025112346556672
```

After recover `x_l` and `y_l`, I decrypted the `FLAG` by this script:
```python
from hashlib import sha256
from random import randint
from Crypto.Util.number import getPrime, long_to_bytes
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad

a = (a_h << 242) + a_l
b = (b_h << 242) + b_l

iv = bytes.fromhex(iv)
key = sha256(long_to_bytes(pow(a, b, p))).digest()[:16]
cipher = AES.new(key, AES.MODE_CBC, iv)
enc = bytes.fromhex(enc)
print(cipher.decrypt(enc))
```
The `FLAG` is: `HTB{y0u_5h0u1d_h4v3_u53d_c00p325m17h}`

---

> Author: [dasHaus165](https://haopham23.github.io/dashaus165blog/)  
> URL: https://haopham23.github.io/dashaus165blog/elliptic-labyrinth-revenge/  

