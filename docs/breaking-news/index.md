# nullcon HackIM CTF Berlin 2023: breaking news

A writeup on nullcon CTF 2023 - Breaking news
<!--more-->

{{< admonition note "Challenge Information" >}}
* **Given materials:** [Get it here!](https://ctf.nullcon.net/files/3a179c12d2e6aa2d9ce5d57182783e49/breaking_news.zip?token=eyJ1c2VyX2lkIjoxNDkyLCJ0ZWFtX2lkIjpudWxsLCJmaWxlX2lkIjo1OH0.ZAqdbQ.nM7TF5m5P4TFNWdzO8y8JJEbBD0)
* **Description:** Alice started to encrypt the flag, but realised halfway she was unhappy with her key. So she created a new one.
* **Category:** Crypto
{{< /admonition >}}

The encyption code is shown below:

```python
from Crypto.PublicKey import RSA
from secret import flag
from Crypto.Cipher import PKCS1_OAEP
from Crypto.Util.number import inverse
from binascii import hexlify

key1 = RSA.import_key(open('key1.pem','rb').read())
key2 = RSA.import_key(open('key2.pem','rb').read())

msg1 = flag[:len(flag)//2]
msg2 = flag[len(flag)//2:]

cryptor = PKCS1_OAEP.new(key1)
c1 = cryptor.encrypt(msg1)

cryptor = PKCS1_OAEP.new(key2)
c2 = cryptor.encrypt(msg2)

writer = open('ciphers','wb')
writer.write(hexlify(c1) + b'\n' + hexlify(c2))
writer.close()
```
The ciphertexts:
```python
c1 = "3b6ccd7fa1de0455945998bc024adc6c2b60ccc8f020cbf024c3d4f98eafdf6a43afd15ec4d9a32f84cb61f7a5462547f2e3622b547c3e9ccb723102805544b373a80f4d252a1081db6d5c5499b222093fd4bb7997c68ab0ed8a9ac3bd0dae64cdfb946da1e311ef6e216ddf2dac14ea3710d5622269f08073598c24a3000a6dd6270ca0db5c304102bc9a5cd104484a2c0ced339121f13499c795de343ad2e655d4ace726654ee9f110e4bee3db95d8e514bd6e658769a01638ff2e9ce954dc09def3b01f6d598ddae2ca9735c8e8f9b71c96984a114084fb0a25b3646481e8c8d4d8adfedc7afad7be7a009c6d12753db4216ab9fd7fc8c37c819aef6a8bce"
c2 = "998d5eeb0c048ade8cd807cb582b15a7799e8481a7476dbe8e0310b5ffc5161add92539bc0a374333c11f5f2008195782a44e45f2394fe3115af59fbc73ad24c4d084d79ba8e5896b644917335fd9a0e07c1d1d316e50480ba44c67b6fc04a2ce33dbc721768f1f874ff2ce1ec0503a4a7c67d10119ff9f79030459068de24ff24593e16877fd74c5a12d0e64a3d62e61b13c403aad2fe605601e8a097aba99707e305e3125a3c89f3d6beccc2f19a32fdac9ab7df181938b9b80d83a54c9c23ef11affff0fca67ecd9d45c58ece90a44ecd60aff7be05bf97cb554c563a3f9139d99f7c76a07acaaa261b1d6cb41e228fb2aca02ed1c64d468aabb8cfaa9210"
```

It seems that the encryption of the first and the last half of `flag` is not relevant. But at least we should check the public key and the modulus of `key1.pem` and `key2.pem`.

```python
from Crypto.PublicKey import RSA

key1 = RSA.import_key(open('breaking_news\key1.pem','rb').read())
key2 = RSA.import_key(open('breaking_news\key2.pem','rb').read())
e1 = key1.e
e2 = key2.e
n1 = key1.n
n2 = key2.n
```

```terminal
e1 (bit_length = 2045): 2726678763069742290493975678771854042946997886526658885273589794982550850524484749274268692898909814780790152954912126546903921714367517303032050244891472514523849708519935681313568761427297382724990480371626349446130805480456004417663238567505831802110513475009108333474503714942886902760554023856774744528158730164306665386046820676766827602146514316823937019543742466850364749529076296428648908924805389552335245924389342424467024736023185451341764743413642429057857014342930366786601460889647066379811076861622890568687138322791084455379633576246829677348537342669805283721207716632578516825617674863582898823857
n1 (bit_length = 2048): 22446667384135380364728550075881381726053468195188168013152415796352249451953078762739428049146187827398120100488504055606635548019009256889092903583429161620832130884808020463159053237738268615549212998960185401460575049040129569674912795281010462969369309454450379821434551440286230754665181962106098655531061808191857271372118647412341530534507363035768520018955713235360762429287435263772391386477249961479679573552073377992778864273735159890546438757223394182339510136036600928909879322891169743059815434712294601960066048614718984000476293007036534025674690519367295530441293542361110626472198678166093083778539
e2 (bit_length = 2047): 10256146915389376804733503849378394040226738315438048733908212356423497844689763806899851419713477281451672208599322793619218516307544360848813105274673995810641694951995105185766820550758709887004645074369214964198132251828883683227550638515829482479043992765245534290447294029738255137528007434087457833764737212244291686107900819383665191108248293367869921932616558880407836029541291465648421673321173968994233210416174380468374540776778309365619130742370523864457591882618770539218576302746506646714903352082566770512709919759453487307052199672663455436660181653236413741514850568323328630717846871365400337385289
n2 (bit_length = 2048): 22446667384135380364728550075881381726053468195188168013152415796352249451953078762739428049146187827398120100488504055606635548019009256889092903583429161620832130884808020463159053237738268615549212998960185401460575049040129569674912795281010462969369309454450379821434551440286230754665181962106098655531061808191857271372118647412341530534507363035768520018955713235360762429287435263772391386477249961479679573552073377992778864273735159890546438757223394182339510136036600928909879322891169743059815434712294601960066048614718984000476293007036534025674690519367295530441293542361110626472198678166093083778539
```

There are two things I noticed from these parameters:
  1. They share the same modulus. This means the fall of one encryption will maybe affect the other one.
  2. Their public keys are extremely big. Practically, the public keys are just small numbers (such as 655347) for the performance and security. Big public keys often lead to small private keys, make them vulnerable to the Wierner attack.

I will break them one by one.
## The first half

I tried and happily it's broken by the Wiener Attack. For the Wiener Attack, readers should read [this](https://cryptohack.gitbook.io/cryptobook/untitled/low-private-component-attacks/wieners-attack).

Here is the code:

```python
import owiener
N = n1
try:
    d1 = owiener.attack(e1, N)
    k1 = RSA.construct((N, e1, d1))
    cryptor1 = PKCS1_OAEP.new(k1)
    c1 = bytes.fromhex(c1)
    print(cryptor1.decrypt(c1))
except:
    print("Failed")
```
The library `owiener` is in [this repo](https://github.com/orisano/owiener).

Result:
```
b'ENO{n3ver_reus3_your_pr1mes_4_a_'
```
## The last half
The last half is not that easy. I tried the same attack and it failed. But maybe we can exploit some properties from known key `(e1, d1)` in the previous section. Fortunately, it does! Based on [this paper](http://www.ams.org/notices/199902/boneh.pdf) of **Prof. Dan Boneh**, the factorization of $N$ can be computed in polynomial time when we're given one pair `(e, d)`. 

One implementation of this fast-factor algorithm is available in [this repo](https://gist.github.com/AntonKueltz/73c28f5a2ceb5f37a3db471068a36a68). I made a tiny optimization in operator to make it works with big numbers, you can try to guess what it is, hehe. 

Here is the code:
```python
p, q = factor(N, e1, d1)
phi = (p - 1) * (q - 1)
d2 = inverse(e2, phi)
k2 = RSA.construct((N, e2, d2))
cryptor2 = PKCS1_OAEP.new(k2)
print(cryptor1.decrypt(c1) + cryptor2.decrypt(c2))
```
The final result:
```
b'ENO{n3ver_reus3_your_pr1mes_4_a_new_k3y_you_have_2_p4y_th3_pr1ce}'
```
## Conclusion
This challange is very helpful to me, because it contains many (2) attacks that I forgot. It's a nice chance to review RSA!


---

> Author: [dasHaus165](https://haopham23.github.io/dashaus165blog/)  
> URL: https://haopham23.github.io/dashaus165blog/breaking-news/  

