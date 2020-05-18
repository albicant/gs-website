---
title: Implementing a Feistel cipher with Python
date: 2020-03-21 19:26:01
tags:
---
### Background
During my Winter 2020 term at [Portland State University](https://www.pdx.edu/cecs/home), I completed the [CS485: Cryptography](https://www.pdx.edu/computer-science/cs485) elective with [Dr. Sarah Mocas](http://web.cecs.pdx.edu/~sarah/). During this course, I had a chance to gain hands-on expirience implementing two cryptographic algorithms: a [Feistel cipher](https://en.wikipedia.org/wiki/Feistel_cipher) and the [ElGamal encryption](https://en.wikipedia.org/wiki/ElGamal_encryption) algorithm. In this post, I would like to share the details of my implementation of a Feistel cipher using a 64 bit block size and a 64 bit key using [Python 3](https://www.python.org/).

### Implementation
I chose Python because of its native support of large numbers, as well as very easy conversion between decimal and hexidecimal numbers. Officiency of the implementation was not an issue due to it being only an educational project to help students gain better understanding of cryptography.

The cipher consist of 3 major steps:

### Step 1: Key generation
The key generation algorithm works as follows:
* Uses the 64 bit secret key __K__ (8 bytes)
* Left rotates __K__ by 1 bit 192 times (64 * 3)
* Creates 16 new keys of consisting of 12 bytes

``` Python
    def shiftBitLeft(self, block):
        if block >= self.constant_64bit:
            block -= self.constant_64bit
            block = block << 1
            block += 1
        else:
            block = block << 1
        return block

    def createKeyArray(self, block):
        key_array = []
        for i in range(0, 64):
            block = self.shiftBitLeft(block)
            key_array.append(block)

        return key_array

    def generateKeys(self, key):
        row_keys = self.createKeyArray(key)
        keys = []
        counter = -1
        x = 0
        while(x < 192 and counter < 16):
            i = x % 64
            byte_arr = self.breakHexIntoChunks(row_keys[i])
            if(x % 12 == 0):
                keys.append([])
                counter += 1

            b = x % 4
            if counter % 2 == 1:
                b += 4
            keys[counter].append(int(byte_arr[b], 16))
            x += 1
        
        return keys
```


### Step 2: Encryption
* Input whitening step: 
  * The input is the 64-bit block divided into 4 words __w<sub>0</sub>, w<sub>1</sub>, w<sub>2</sub>, w<sub>3</sub>__.  
  * XOR each word with 16 bits of the key __K = K<sub>0</sub>K<sub>1</sub>K<sub>2</sub>K<sub>3</sub>__.  
  * The output is     
    * __R<sub>0</sub> = w<sub>0</sub> ⊕ K<sub>0</sub>, R<sub>01</sub> = w<sub>1</sub> ⊕ K<sub>1</sub>,__
    * __R<sub>2</sub> = w<sub>2</sub> ⊕ K<sub>2</sub>, R<sub>3</sub> = w<sub>3</sub> ⊕ K<sub>3</sub>.__
* Set the round number to __round=0__. After each of the 16 rounds increment __round__ by one.
* Compute __F(R<sub>0</sub>, R<sub>1</sub>, round)__. This function returns two values __F<sub>0</sub>__ and __F<sub>1</sub>__.  
    * Compute __R<sub>2</sub> ⊕ F<sub>0</sub>__. This value is __R<sub>0</sub>__ for the next round.  
    * Compute __R<sub>3</sub> ⊕ F<sub>1</sub>__. This value is __R<sub>1</sub>__ for the next round.   
    * The value __R<sub>0</sub>__ becomes __R<sub>2</sub>__ for the next round. 
    * The value __R<sub>1</sub>__ becomes __R<sub>3</sub>__ for the next round.
    * Repeat for 16 rounds (but not the whitening step this only happens at the start of the first round).  
* After the last of the 16 rounds: 
    * Undo the last swap by setting 
    __y<sub>0</sub>__ to __R<sub>2</sub>__ from the end of the 16th round;
    __y<sub>1</sub>__ to __R<sub>3</sub>__ from the end of the 16th round;
    __y<sub>2</sub>__ to __R<sub>0</sub>__ from the end of the 16th round and 
    __y<sub>3</sub>__ to __R<sub>1</sub>__ from the end of the 16th round.
* Output whitening step: 
	* XOR each word with 16 bits of the key __K = K<sub>0</sub>K<sub>1</sub>K<sub>2</sub>K<sub>3</sub>__.  
The input is __y<sub>0</sub>y<sub>1</sub>y<sub>2</sub>y<sub>3</sub>__. The output is     
    * __C<sub>0</sub> = y<sub>0</sub> ⊕ K<sub>0</sub>, C<sub>01</sub> = y<sub>1</sub> ⊕ K<sub>1</sub>,__
    * __C<sub>2</sub> = y<sub>2</sub> ⊕ K<sub>2</sub>, C<sub>3</sub> = y<sub>3</sub> ⊕ K<sub>3</sub>.__
The value __C<sub>0</sub>C<sub>1</sub>C<sub>2</sub>C<sub>3</sub>__ is the ciphertext.

``` Python
    def whitening(self, word, key):
        r = []

        key = format(key, "016x")
        for i in range(0, 4):
            j = i * 4

            sw = word[j: j + 4]
            sk = key[4 * i:4 * i + 4]
            w = int(sw, 16) ^ int(sk, 16)
            r.append(w)
        return r

    def g_function(self, r, k0, k1, k2, k3):
        r = format(r, '04x')
        g1 = int(r[:2], 16)
        g2 = int(r[2:4], 16)
        g3 = int(self.ftable[g2 ^ k0]) ^ g1
        g4 = int(self.ftable[g3 ^ k1]) ^ g2
        g5 = int(self.ftable[g4 ^ k2]) ^ g3
        g6 = int(self.ftable[g5 ^ k3]) ^ g4
        return int(format(g5, '02x') + format(g6, '02x'), 16)

    def f_function(self, r0, r1, round):
        k = self.keys[round]

        t0 = self.g_function(r0, k[0], k[1], k[2], k[3])
        t1 = self.g_function(r1, k[4], k[5], k[6], k[7])

        k89 = int(format(k[8], '02x') + format(k[9], '02x'), 16)
        kab = int(format(k[10], '02x') + format(k[11], '02x'), 16)
        f0 = (t0 + 2 * t1 + k89) % self.const_2_16
        f1 = (2 * t0 + t1 + kab) % self.const_2_16
        return [f0, f1]

    def round_function(self, r, round):
        f = self.f_function(r[0], r[1], round)

        new_r0 = r[2] ^ f[0]
        new_r1 = r[3] ^ f[1]
        new_r2 = r[0]
        new_r3 = r[1]
        return [new_r0, new_r1, new_r2, new_r3]

def encrypt(self, pt_hex):
        r = self.whitening(pt_hex, self.original_key)

        for i in range(0, 16):
            r = self.round_function(r, i)

        y = [r[2], r[3], r[0], r[1]]
        y_hex = ""
        for yi in y:
            y_hex += format(yi, "04x")

        c = self.whitening(y_hex, self.original_key)
        c_str = ""
        for ci in c:
            c_str += format(ci, "04x")
        return c_str
```

### Step 3: Decryption
The algorithm to decrypt is the same as the algorithm to encrypt except that the keys are generated and used in reverse order.
``` Python
    def decrypt(self, ct_hex):
        r = self.whitening(ct_hex, self.original_key)
        for i in range(15, -1, -1):
            r = self.round_function(r, i)

        p = [r[2], r[3], r[0], r[1]]
        p_hex = ""
        for pi in p:
            p_hex += format(pi, "04x")

        p = self.whitening(p_hex, self.original_key)
        p_str = ""
        for pi in p:
            p_str += format(pi, "04x")
        return p_str
```

### Further Reading
You can find the full code of my implementation [here](https://github.com/albicant/CS485_feistel).