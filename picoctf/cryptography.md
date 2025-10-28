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

## References
- https://play.picoctf.org/practice?category=2&page=1&search=miniRSA
- https://en.wikipedia.org/wiki/RSA_cryptosystem#Operation
- https://www.geeksforgeeks.org/computer-networks/rsa-algorithm-cryptography/
------------------------------------------------------------------------------------------------------------------------------------
