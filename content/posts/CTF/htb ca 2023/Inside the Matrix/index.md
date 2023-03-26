---
title: "Cyber Apocalypse 2023: Inside the matrix"
subtitle:
date: 2023-03-26T10:15:56+07:00
draft: false
description:
keywords:
license:
comment: false
weight: 0
tags:
  - writeups
  - crypto
  - htb
categories:
  - Writeups
hiddenFromHomePage: false
hiddenFromSearch: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: true
lightgallery: false
password:
message:
# See details front matter: https://fixit.lruihao.cn/documentation/content/#front-matter
---
A writeup on Inside the matrix
<!--more-->
{{< admonition note "Challenge Information" >}}
* **Given materials:** [Get it here!](https://drive.google.com/file/d/1w3gAAQ9VKg6HucPePcDwvUqzlKTWaRbk/view?usp=sharing)
* **Description:** As you deciphered the Matrix, you discovered that the astronomy scientist had observed that certain stars were not real. He had created two 5x5 matrices with values based on the time the stars were bright, but after some time, the stars stopped emitting light. Nonetheless, he had managed to capture every matrix until then and created an algorithm that simulated their generation. However, he could not understand what was hidden behind them as he was missing something. He believed that if he could understand the stars, he would be able to locate the secret tombs where the relic was hidden.
* **Category:** Crypto - Easy
{{< /admonition >}}

The server script is shown below:

```python
from sage.all_cmdline import *
# from utils import ascii_print
import os

FLAG = b"HTB{????????????????????}"
assert len(FLAG) == 25


class Book:

    def __init__(self):
        self.size = 5
        self.prime = None

    def parse(self, pt: bytes):
        pt = [b for b in pt]
        return matrix(GF(self.prime), self.size, self.size, pt)

    def generate(self):
        key = os.urandom(self.size**2)
        return self.parse(key)

    def rotate(self):
        self.prime = random_prime(2**6, False, 2**4)

    def encrypt(self, message: bytes):
        self.rotate()
        key = self.generate()
        message = self.parse(message)
        ciphertext = message * key
        return ciphertext, key


def menu():
    print("Options:\n")
    print("[L]ook at page")
    print("[T]urn page")
    print("[C]heat\n")
    option = input("> ")
    return option


def main():
    book = Book()
    ciphertext, key = book.encrypt(FLAG)
    page_number = 1

    while True:
        option = menu()
        if option == "L":
            # ascii_print(ciphertext, key, page_number)
            print(ciphertext, key, page_number)
        elif option == "T":
            ciphertext, key = book.encrypt(FLAG)
            page_number += 2
            print()
        elif option == "C":
            print(f"\n{list(ciphertext)}\n{list(key)}\n")
        else:
            print("\nInvalid option!\n")


if __name__ == "__main__":
    try:
        main()
    except Exception as e:
        print(f"An error occurred: {e}")
```
## Problem statement
The code defines a class `Book` that is used to generate a key matrix and encrypt a message using matrix multiplication. The matrix is generated randomly each time a message is encrypted, and its size is fixed at $5\times 5$. The program encrypts a flag, stored in `FLAG`, using the `Book` class and presents a menu to the user to interact with the encrypted flag.

The main function of the code presents a menu to the user with three options:

- `[L]ook at page`: displays the ciphertext and key matrix for the current page number. Here is an example output when you choose this option:
```
Options:

[L]ook at page
[T]urn page
[C]heat

> L

          _________   _________
   ______/        5\ /       6 \_______
 /| --------------- |  --------------- |\
|| Ciphertext:--- - | Key:------------ |||
|| ---------------- | ------  -------- |||
|| ---------- ----- | ---------------- |||
|| [3,12,21,20,8]-- | [18,18,21,26,24] |||
|| [1,1,9,7,1]----- | [21,7,10,9,2]--- |||
|| [10,3,8,6,13]--- | [22,1,24,22,12]- |||
|| [0,19,24,15,12]- | [7,21,7,20,2]--- |||
|| [10,4,6,2,4]---- | [26,25,17,3,25]- |||
|| ---------------- | ------ ----- --- |||
|| --- - ---------- | ---------------- |||
||______________ _  |  ________________|||
L/______/---------\\_//W--------\_______\J
``` 

- `[T]urn page`: generates a new key matrix and ciphertext for the next page number.

- `[C]heat`: displays the ciphertext and key matrix in list type. The Cheat output of above example page is:
```
[(3, 12, 21, 20, 8), (1, 1, 9, 7, 1), (10, 3, 8, 6, 13), (0, 19, 24, 15, 12), (10, 4, 6, 2, 4)]
[(18, 18, 21, 26, 24), (21, 7, 10, 9, 2), (22, 1, 24, 22, 12), (7, 21, 7, 20, 2), (26, 25, 17, 3, 25)]
```
## Initial analysis
### Primes
Prime $p$ is changed whenever `Turn page` option is chosen. Though we don't know what $p$ is, we know that it would be from 16 to 64. There are  12 primes in this range, which are $17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61$.

### The encryption
It's just a multiplication between two $5 \times 5$ matrixs over the field of integers modulo $p$: 


$$C \equiv M\times K (\text{mod } p)$$

$$\Leftrightarrow M \equiv C\times K^{-1}  (\text{mod } p)$$



### Conclusion
We already have key $K$ and ciphertext $C$ by using `Cheat option`. Then if we know $p$, we can easily recover message $M$ in modulus $p$. Because $p$ is changeable, we can gather several pairs $(M_i, p_i)$ where $i \geq 2$.
## Solution method
Suppose there are some entries in a key $K_1$ which are larger than 59, then $p_1$ must be 61. 

Suppose all entries in a key $K_2$ are smaller than 17, then it's likely that $p_2$ is 17.

If we have $K_1$ and $K_2$, then we can recover $M_1$, $M_2$. By applying CRT (Chinese Remainder Theorem) for 2 pairs $(M_1, 61)$ and $(M_2, 17)$, we can get $M$ in modulus $61\times 17 = 1037$. Because every entries of the actual message's matrix are bytes, they would be smaller than 128 (which is much smaller than 1037). This means our $M$ is actually the message itself.

So our mission is just to find $C_1, C_2$ by using `Turn page` many times. Here is the script after we gather enough materials (I used $p_1=61$ and $p_2=19$):

```python
M_1 = [11, 23, 5, 1, 47, 48, 48, 46, 34, 3, 55, 34, 55, 43, 51, 34, 54, 55, 52, 53, 54, 33, 33, 33, 3]
M_2 = [15, 8, 9, 9, 13, 10, 10, 12, 0, 7,2, 0, 17, 9, 13,0, 1, 2, 14, 0,1, 14, 14, 14, 11]
res = []
from sympy.ntheory.modular import crt
for i in range(len(M_1)):  
    m = [61, 19]
    v = [M_1[i], M_2[i]]
# Use crt() method 
    crt_m_v = crt(m, v)[0]
    res.append(crt_m_v)
print(''.join([chr(x) for x in res]))
```

## Results
The Flag: `HTB{l00k_@t_7h3_st4rs!!!}`
