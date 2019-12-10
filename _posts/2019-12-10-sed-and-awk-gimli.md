---
layout: post
title:  "Porting test vectors with sed and awk"
date:   2019-12-10 20:13:00 +0100
---
# Intro
Lately I've been working on a port of some C reference crypto code to rust for the `gimli` lightweight cipher and I wanted to make a quick post on some sed and awk which made helped me make part of the test suite.

## Test vectors
When working with an algorithm which you don't understand it's critical to have a set of tests to let validate your code. In crypto terminology you have what are called `test vectors`. Each vector defines a set of inputs and an expected output i.e. `f(x1, x2, x3 ...) = y`. In my case I had a file of the form    

```  
Count = 1
Key = 000102030405060708090A0B0C0D0E0F101112131415161718191A1B1C1D1E1F
Nonce = 000102030405060708090A0B0C0D0E0F
PT =
AD =
CT = 14DA9BB7120BF58B985A8E00FDEBA15B

Count = 2
Key = 000102030405060708090A0B0C0D0E0F101112131415161718191A1B1C1D1E1F
Nonce = 000102030405060708090A0B0C0D0E0F
PT =
AD = 00
CT = E8D50453F84B575412327D7C0302D8D3
...
```
The fields are the `Count` which indexes the test vector, the `Key` and `Nonce` which are constant for all vectors, the plaintext or `PT`, the associated data or `AD` and the ciphertext or `CT`. In this case the equality looks something like `gimli(PT, AD, nonce, key) = CT`.
The total count was 1089. Given the functions that I had made I wanted each of these available as a vector of `u8` values in my rust code.

The source of these test vectors can be seen in the path `gimli/Implementations/crypto_aead/gimli24v1/LWC_AEAD_KAT_256_128.txt` of the archive [gimli.zip](https://csrc.nist.gov/CSRC/media/Projects/lightweight-cryptography/documents/round-2/submissions-rnd2/gimli.zip)

## The goal
What I need to end up with is a collection of lines of the form
```
(vec![], vec![], vec![0x14,0xDA,0x9B,0xB7,0x12,0x0B,0xF5,0x8B,0x98,0x5A,0x8E,0x00,0xFD,0xEB,0xA1,0x5B]),
(vec![], vec![0x00], vec![0xE8,0xD5,0x04,0x53,0xF8,0x4B,0x57,0x54,0x12,0x32,0x7D,0x7C,0x03,0x02,0xD8,0xD3]),
...
```
which is directly consumable and allows for a simple test loop
```
for vec in cipher_vectors.iter(){
    assert_eq!(vec.2, gimli_aead_encrypt(&vec.0, &vec.1, &nonce, &key));
    assert_eq!(vec.0, gimli_aead_decrypt(&vec.2, &vec.1, &nonce, &key));
}
```

## Transformations
Lets say that the original file is simply called `vecs.txt`. How do I get to our goal without tedious manual edits? First off remove the lines I don't need.  
`Count` is unnecessary so  
```
sed -i '/Count/d' vec.txt
```
`Key` and `Nonce` are constant and so can be removed.
```
sed -i '/Key/d' vec.txt
sed -i '/Nonce/d' vec.txt
```
Similarly those blank lines aren't of any use
```
sed -i '/^\s*$/d' vec.txt
```
At this point our file is of the form
```
PT =
AD =
CT = 14DA9BB7120BF58B985A8E00FDEBA15B
PT =
AD = 00
CT = E8D50453F84B575412327D7C0302D8D3
...
```
Next lets add the `vec![]` macro by matching on the `=` and the end of line.
```
sed -i 's/= /= vec![/' vec.txt
sed -i 's/$/]/' vec.txt
```
Yielding
```
PT = vec![]
AD = vec![]
CT = vec![14DA9BB7120BF58B985A8E00FDEBA15B]
PT = vec![]
...
```
Now remove the first 5 characters of each line.
```
sed -i 's/^.....//' vec.txt
```
Resulting in
```
vec![]
vec![]
vec![14DA9BB7120BF58B985A8E00FDEBA15B]
...
```
I'd like to convert that hex into `0x` rust hex notation and I can do that with the following
```
sed -i 's/\([A-Z0-9][A-Z0-9]\)/0x\1, /g' vec.txt
```
What's happening here is a match group `\1` is being created on any matching of two elements with the upper alphanumeric character set. That match is prepended with `0x` and appended with a `,` which gives
```
vec![]
vec![]
vec![0x14, 0xDA, 0x9B, 0xB7, 0x12, 0x0B, 0xF5, 0x8B, 0x98, 0x5A, 0x8E, 0x00, 0xFD, 0xEB, 0xA1, 0x5B, ]
...
```
I've got extra commas and spaces at the end of each line, but rust is tolerant of that so I'll leave it. I could clean up each line easily enough if desired, but I'll leave that to the reader.

Now I need to compress three lines into one. I'm not aware of a simple sed method to do this, however awk is perfect for this.
```
awk '{ printf "%s", $0; if (NR % 3 == 0) print ""; else printf " " }' vec.txt > vecs.txt
```
What's happening here is that awk is reading our input file `vec.txt` and printing each line without a newline character unless that line is modulo 3. I put this into a new file `vecs.txt` which has the form
```
vec![] vec![] vec![0x14, 0xDA, 0x9B, 0xB7, 0x12, 0x0B, 0xF5, 0x8B, 0x98, 0x5A, 0x8E, 0x00, 0xFD, 0xEB, 0xA1, 0x5B, ]
vec![] vec![0x00, ] vec![0xE8, 0xD5, 0x04, 0x53, 0xF8, 0x4B, 0x57, 0x54, 0x12, 0x32, 0x7D, 0x7C, 0x03, 0x02, 0xD8, 0xD3, ]
...
```
A bit more cleanup
```
sed -i 's/ vec!/,vec!/g' vecs.txt
sed -i 's/, /,/g' vecs.txt
sed -i 's/$/)/' vecs.txt
sed -i 's/^/(/' vecs.txt
```
and now I've have our desired format
```
(vec![],vec![],vec![0x14,0xDA,0x9B,0xB7,0x12,0x0B,0xF5,0x8B,0x98,0x5A,0x8E,0x00,0xFD,0xEB,0xA1,0x5B,])
(vec![],vec![0x00,],vec![0xE8,0xD5,0x04,0x53,0xF8,0x4B,0x57,0x54,0x12,0x32,0x7D,0x7C,0x03,0x02,0xD8,0xD3,])
...
```
wrap the whole thing in one more `vec![]` and I've got an iterable test vector.


# Conclusion
What's taken place here is a series of simple text transformations which has taken a file of the format
```
Count = 1
Key = 000102030405060708090A0B0C0D0E0F101112131415161718191A1B1C1D1E1F
Nonce = 000102030405060708090A0B0C0D0E0F
PT =
AD =
CT = 14DA9BB7120BF58B985A8E00FDEBA15B
```
to
```
(vec![],vec![],vec![0x14,0xDA,0x9B,0xB7,0x12,0x0B,0xF5,0x8B,0x98,0x5A,0x8E,0x00,0xFD,0xEB,0xA1,0x5B,])
```
We've taken structured, but useless plaintext and converted it into a structure directly usable in a test suite. In fact you can see this code in my [gimli repo](https://github.com/darakian/gimli).

I hope this post has been helpful.

```
Thanks for reading.
```
