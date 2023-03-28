# Cyber Apocalypse 2023: Elliptic Labyrinth

A writeup on Elliptic Labyrinth
<!--more-->

{{< admonition note "Challenge Information" >}}
* **Given materials:** [Get it here!](https://drive.google.com/file/d/1w4QyL7cKzhcZJ_qqakudh6fXLtj8mk6p/view?usp=sharing)
* **Description:** 
* **Category:** Crypto - Medium
{{< /admonition >}}

The server script is shown below:
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
    print("2. Get point in path")
    print("3. Try to exit the labyrinth")
    option = input("> ")
    return option


def main():
    ec = ECC(512)

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
            A = ec.gen_random_point()
            print(json.dumps({'x': hex(A[0]), 'y': hex(A[1])}))
        elif choice == '3':
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

  - Option 2: Obtain a random point on the curve.

  - Option 3: Provide the encrypted FLAG.

Our mission is to decrypt the flag.
## Initial analysis
### What we need to decrypt the flag?
Obviously, we cannot break the AES to find the flag without the `key`. To recover the `key`, we need to know all elliptic curve's parameters, which are `a`, `b` and `p`. We already known `p`, so what we do is trying to retrieve `a` and `b` from the information provided by the server.

### Having many points on the curve
Every point $P(x, y)$ belonging to this elliptic curve must satisfy the equation: $y^2 \equiv x^3 + ax + b (\text{mod } p)$. To find `a` and `b` in `p`, we must at least have a system of 2 equations like this. Fortunately, the server allows user to generate many points.

## Solution method
Suppose we have two different points $M(x_m, y_m)$, $N(x_n, y_n)$ in the curve. We recover $a,b$ by below formulas:

$a \equiv (y^2_m - y^2_n - (x^3_m - x^3_n))(x_m - x_n)^{-1} (\text{mod } p)$

$b \equiv y^2_m - x^3_m - ax_m (\text{mod } p)$

The script:
```python
def recover(p, M, N):
    x1, y1 = M
    x2, y2 = N
    a = pow(x1 - x2, -1, p) * (pow(y1, 2, p) - pow(y2, 2, p) - (pow(x1, 3, p) - pow(x2, 3, p))) % p
    b = (pow(y1, 2, p) - pow(x1, 3, p) - a * x1) % p
    return a, b
```

That's all! By having `a` and `b`, we can easily recover the `key` and therefore decrypt the FLAG:
```python
from hashlib import sha256
from random import randint
from Crypto.Util.number import getPrime, long_to_bytes
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad

iv = bytes.fromhex(iv)
key = sha256(long_to_bytes(pow(a, b, p))).digest()[:16]
cipher = AES.new(key, AES.MODE_CBC, iv)
enc = bytes.fromhex(enc)
print(cipher.decrypt(enc))
```

## Results
The flag is `HTB{d3fund_s4v3s_th3_d4y!}`

---

> Author: [dasHaus165](https://haopham23.github.io/dashaus165blog/)  
> URL: https://haopham23.github.io/dashaus165blog/elliptic-labyrinth/  

