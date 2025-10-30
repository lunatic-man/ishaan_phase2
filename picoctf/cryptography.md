# RSA_Oracle
In this challenge, we were given two input files and we had to find out the message hidden in the `secret.enc` file. 

## Solution 
To solve this challenge, I followed the steps as given below: 
- This challenge took a long while to solve as I had to understand how the RSA worked, and how exactly did the Chose Plaintext Attack method work.
- I went to youtube and I learned how these encryptions worked and I did go on a tangent which is mentioned in notes. Approaching my mentor I got the idea to go deeper into CPA.
- Learning about CPA, I realised that we could generate the password for a random digit, multiply the ciphertext with the ciphertext of the random digit, and then decrypt the ciphertext to get the final value. It was confusing but I will elaborate each step here.
- First we pick a number of our choosing, I chose 2 because of its simplicity, but you can choose any number. Doing this, I encrypted the number to get a ciphertext.
- I then multiplied this ciphertext with the `password.enc` ciphertext. This gave me a new ciphertext, let's call it final ciphertext.
- Putting this ciphertext into the decryption oracle, I got a hex value. Now this hex value is 2 * password(in some format).
- So I converted this hex value back into decimal so that I could remove the 2. After removing the 2, I got a decimal number.
- Taking this decimal number, I converted it to Hex, and then converting this hex back to ASCII, I got the password.
- This was done using a script, which is shown below:
```bash
11:19:01 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~  → nano exploit.py
  GNU nano 7.2                                                   exploit.py                                                            
#!/usr/bin/env python3
from pwn import *

target = remote('titan.picoctf.net', 54224)
data = target.recvuntil('decrypt.')
print(data.decode())

input = (b'E') + (b'\n')
target.send(input)

data = target.recvuntil('keysize):')
print(data.decode())

input = p8(0x02) + (b'\n')
target.send(input)
data = target.recvuntil('ciphertext (m ^ e mod n)')
data = target.recvline()
result=int(data.decode())*176503704976404772434811463447365873483049085206606134568691636565861819498109721675092942173481291168043464>

data = target.recvuntil('decrypt.')
print(data.decode())
input = (b'D') + (b'\n')
target.send(input)

data = target.recvuntil('decrypt:')
print(data.decode())
target.send(str(result)+'\n')

data = target.recvuntil('hex (c ^ d mod n):')
print(data.decode())
data = target.recvline()
print(data.decode())


^G Help         ^O Write Out    ^W Where Is     ^K Cut          ^T Execute      ^C Location     M-U Undo        M-A Set Mark
^X Exit         ^R Read File    ^\ Replace      ^U Paste        ^J Justify      ^/ Go To Line   M-E Redo        M-6 Copy
```
- The conversion of Hex to Decimal, then removing 2 from Decimal, then converting Decimal to Hex, and finally Hex to ASCII was done using online tools. This gave me the password as  `881d9`
![image of final conversion of Hex to ASCII](/images/cryptography/Screenshot-2025-10-29-11-40-38.png)

- Now this was half of the challenge, the second half was using this password to decrpt the secret.enc.
- Using the hint provided in the challenge, I crafted a command, `openssl enc -aes-256-cbc -d -in secret.enc -k 881d9` to get the final flag.
```bash
11:31:14 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → openssl enc -aes-256-cbc -d -in secret.enc -k 881d9
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
picoCTF{su((3ss_(r@ck1ng_r3@_881d93b6}11:31:15 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 

````

## Flag

`picoCTF{su((3ss_(r@ck1ng_r3@_881d93b6}` 

## Notes 
The tangent I went on was assuming that we could use a wordlist to crack the password. Not really sure if it is feasible or not, but some research has told me it is not doable. Not entirely convinced so will go deeper down this rabbit hole to figure it out.

## What I learned 
#### Various methods of cracking an RSA
This method actually introduced me to a lot of other methods which can be used depending on what resources you have access to when doing an RSA challenge.

#### CPA Attack
The Chosen Plaintext Attack method was pretty crazy, understanding it took a while, basically you multiply the password ciphertext with another ciphertext of your choosing, decrpyt the said ciphertext, get the hex value, convert to dec, remove the number and then convert the decimal number to hex and then to ASCII to get the password.

#### Openssl Encryption
I did not know that the `openssl` command was capable of encrypting the files using password, so that was also new to me.

## References
- https://www.youtube.com/watch?v=cC8kKIvve-M
- https://www.youtube.com/watch?v=wwkhSL5QWQc
- https://play.picoctf.org/practice/challenge/422?category=2&page=1&search=Oracle
----------------------------------------------------------------------------------------------------------------------------------
# Custom Encryption
To solve this challenge, we had to understand how the encryption mechanism worked and then write our own reverse program.

## Solution 
To solve this challenge, I followed the steps as listed below: 
- I first read through the challenge statement to understand what they wanted us to do. After reading through the challenge statement, I learned that we needed to build a program which does the exact opposite of whatever encryption was applied.
- I then went on to read the code to understand what was actually happening. In this we had defined various programs such as `generator`, `encrypt`, `is_prime`, `dynamic_xor_encrypt` and `test`.
- Now on reading these programs, one can conclude that this is actually a multistep encryption so you had to make individual programs to reverse them. So I reversed `encypt` using the program as shown below:
```bash
def decrypt(ciphertext, key):
    plain = ""
    for num in ciphertext:
        plain += chr(int((num/ key /311)))
    return plain
```
- Working on reversing the XOR, I used the following program:
```bash
def dynamic_xor_decrypt(ciphertext, text_key):
    plain_text = ""
    key_length = len(text_key)
    for i, char in enumerate(ciphertext):
        key_char = text_key[i % key_length]
        decrypted_char = chr(ord(char) ^ ord(key_char))
        plain_text += decrypted_char
    return plain_text
```
- Adding both these programs at the respective places, and changing the values of `a` and `b` along with copying the `message` array from the `enc_flag` file, we get the result as we wanted.
```bash
20:10:23 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~  → ./exploit.py
a = 89
b = 27
plain is: picoCTF{custom_d2cr0pt6d_dc499538}
```

## Flag
`picoCTF{custom_d2cr0pt6d_dc499538}`

## Notes
This challenge was a mid range one, better be careful while designing reverse algorithms and not mess up. 

## What I learned 
This challenge served as a good practice for the Python language as I was just starting to learn it, a good few functions like `ord()`, along with the change in syntax was new, but it was solvable and doable. Overall a fun challenge. 

## References
- https://play.picoctf.org/practice/challenge/412?category=2&page=1&search=custom
- https://www.w3schools.com/python/
----------------------------------------------------------------------------------------------------------------------------------
# miniRSA
To solve this challenge, we had to decrypt the algorithm and get the flag. 

## Solution 
To solve this challenge, I followed the steps as listed below: 
- I first downloaded the ciphertext that was given in the challenge. On opening it, I found that we had something called as N, e and C. I had no clue about this, but thankfully, I knew the name of the challenge. Knowing this, I went online and google about RSA algorithm.
```bash
14:48:28 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → cat ciphertext

N: 29331922499794985782735976045591164936683059380558950386560160105740343201513369939006307531165922708949619162698623675349030430859547825708994708321803705309459438099340427770580064400911431856656901982789948285309956111848686906152664473350940486507451771223435835260168971210087470894448460745593956840586530527915802541450092946574694809584880896601317519794442862977471129319781313161842056501715040555964011899589002863730868679527184420789010551475067862907739054966183120621407246398518098981106431219207697870293412176440482900183550467375190239898455201170831410460483829448603477361305838743852756938687673
e: 3

ciphertext (c): 2205316413931134031074603746928247799030155221252519872650073010782049179856976080512716237308882294226369300412719995904064931819531456392957957122459640736424089744772221933500860936331459280832211445548332429338572369823704784625368933 
14:48:34 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 
```

- I learned how RSA encryption works by reading up on a website called geeksforgeeks.org. That website beautifully explained this challenge, which helped my solve it quickly.
- I then went online and searched for RSA Decoder. Finding the best website and putting in the values of C, e, and N, I got the flag.
![Image of RSA Decoder](/images/cryptography/Screenshot-2025-10-28-15-01-00.png)

## Flag

`picoCTF{n33d_a_lArg3r_e_ccaa7776}`

## What I learned
#### RSA working
From this challenge, I learned how we go about creating an RSA encryption and what are the `e`, `M`, `C`, `d` & `N` values. This was a very fun challenge, where I learned a lot of new things.

## References
- https://play.picoctf.org/practice?category=2&page=1&search=miniRSA
- https://en.wikipedia.org/wiki/RSA_cryptosystem#Operation
- https://www.geeksforgeeks.org/computer-networks/rsa-algorithm-cryptography/
------------------------------------------------------------------------------------------------------------------------------------
