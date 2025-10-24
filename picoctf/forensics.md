# Trivial Flag Transfer Protocol 
In this challenge, we first need to extract the data from a `.pcapng` file, then decipher the data present in the extracted files and understand what it is finally.

## Solution 
This was easily one of the biggest challenge I have ever solved and it felt like a miniCTF of its own. To solve this, I followed the steps as listed below: 
- Seeing that it was a `.pcapng` extension, I simply put it in Wireshark and first read the protocol heirarchy and then extracted all the objects from the captured packets. I knew how to do this beforehand as I had read a bit about cybersecurity on TryHackMe.
![Protocol Hierarchy image](/images/forensics/Screenshot-2025-10-24-18-28-10.png)
- After extracting the data from wireshark, I saw a few new files which are shown below:
```bash
11:47:05 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → ls
tftp.pcapng
11:48:31 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → wireshark tftp.pcapng & 
[1] 11505
11:48:46 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → file * 
instructions.txt: ASCII text
picture1.bmp:     PC bitmap, Windows 3.x format, 605 x 454 x 24, image size 824464, resolution 5669 x 5669 px/m, cbSize 824518, bits offset 54
picture2.bmp:     PC bitmap, Windows 3.x format, 4032 x 3024 x 24, image size 36578304, resolution 5669 x 5669 px/m, cbSize 36578358, bits offset 54
picture3.bmp:     PC bitmap, Windows 3.x format, 807 x 605 x 24, image size 1466520, resolution 5669 x 5669 px/m, cbSize 1466574, bits offset 54
plan:             ASCII text
program.deb:      Debian binary package (format 2.0), with control.tar.gz , data compression xz
tftp.pcapng:      pcapng capture file - version 1.0
11:50:43 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 
```
- Now in this, we first had to analyze `instructions.txt` and `plan`. Seeing that these are simple ASCII, I simply used the `cat` command on `instructions.txt`. This however gave me a ciphertext, which I decoded with some trial and error and help of cyberchef.io. I ran the ciphers in trial and error on cyberchef.io, with the ROT13 giving me the decoded text. 
```bash
11:50:43 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → cat ins*
GSGCQBRFAGRAPELCGBHEGENSSVPFBJRZHFGQVFTHVFRBHESYNTGENAFSRE.SVTHERBHGNJNLGBUVQRGURSYNTNAQVJVYYPURPXONPXSBEGURCYNA
11:52:51 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 
```
![image of cyberchef.io on ROT13](/images/forensics/Screenshot-2025-10-24-18-25-06.png)

- This told me to read the `plan` file which again turned out to be a ROT13. Decoding it told me that I need to check the images.
- Now using the `file` command on the images, I could see that there was a mismatch in `imagesize` and `cBsize` properties:
```bash
11:52:51 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → file *.bmp
picture1.bmp: PC bitmap, Windows 3.x format, 605 x 454 x 24, image size 824464, resolution 5669 x 5669 px/m, cbSize 824518, bits offset 54
picture2.bmp: PC bitmap, Windows 3.x format, 4032 x 3024 x 24, image size 36578304, resolution 5669 x 5669 px/m, cbSize 36578358, bits offset 54
picture3.bmp: PC bitmap, Windows 3.x format, 807 x 605 x 24, image size 1466520, resolution 5669 x 5669 px/m, cbSize 1466574, bits offset 54
11:58:10 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 
```
- This was in line with the hint given in `plan`. However here I got confused, and I had to go online and google what is `.bmp` extension. This taught me that `.bmp` stands for bitmap and was developed by Windows.
![image of Wiki Search of what is .bmp](/images/forensics/Screenshot-2025-10-24-18-23-13.png)
- Now here I was stuck, and actually went on a tangent, which is mentioned in notes. I completely forgot there was something called as `program.deb` as well which we managed to extract. So reading the file properties of this, I learned that this a debian package, extracting it using the `ar` command, I got two new archives, `control.tar.gz` and `data.tar.xz`
```bash
12:18:06 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → file * 
instructions.txt: ASCII text
picture1.bmp:     PC bitmap, Windows 3.x format, 605 x 454 x 24, image size 824464, resolution 5669 x 5669 px/m, cbSize 824518, bits offset 54
picture2.bmp:     PC bitmap, Windows 3.x format, 4032 x 3024 x 24, image size 36578304, resolution 5669 x 5669 px/m, cbSize 36578358, bits offset 54
picture3.bmp:     PC bitmap, Windows 3.x format, 807 x 605 x 24, image size 1466520, resolution 5669 x 5669 px/m, cbSize 1466574, bits offset 54
plan:             ASCII text
program.deb:      Debian binary package (format 2.0), with control.tar.gz , data compression xz
tftp.pcapng:      pcapng capture file - version 1.0
12:18:08 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → ar x program.deb 
12:18:22 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → file * 
control.tar.gz:   gzip compressed data, max speed, from Unix, original size modulo 2^32 10240
data.tar.xz:      XZ compressed data, checksum CRC32
debian-binary:    ASCII text
instructions.txt: ASCII text
picture1.bmp:     PC bitmap, Windows 3.x format, 605 x 454 x 24, image size 824464, resolution 5669 x 5669 px/m, cbSize 824518, bits offset 54
picture2.bmp:     PC bitmap, Windows 3.x format, 4032 x 3024 x 24, image size 36578304, resolution 5669 x 5669 px/m, cbSize 36578358, bits offset 54
picture3.bmp:     PC bitmap, Windows 3.x format, 807 x 605 x 24, image size 1466520, resolution 5669 x 5669 px/m, cbSize 1466574, bits offset 54
plan:             ASCII text
program.deb:      Debian binary package (format 2.0), with control.tar.gz , data compression xz
tftp.pcapng:      pcapng capture file - version 1.0
12:18:25 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads
```
- Extracting data from these compressions again, we get `control.tar` and `data.tar`

```bash
12:18:25 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → gunzip control.tar.gz; xz -d data.tar.xz
12:18:46 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → file * 
control.tar:      POSIX tar archive (GNU)
data.tar:         POSIX tar archive (GNU)
debian-binary:    ASCII text
instructions.txt: ASCII text
picture1.bmp:     PC bitmap, Windows 3.x format, 605 x 454 x 24, image size 824464, resolution 5669 x 5669 px/m, cbSize 824518, bits offset 54
picture2.bmp:     PC bitmap, Windows 3.x format, 4032 x 3024 x 24, image size 36578304, resolution 5669 x 5669 px/m, cbSize 36578358, bits offset 54
picture3.bmp:     PC bitmap, Windows 3.x format, 807 x 605 x 24, image size 1466520, resolution 5669 x 5669 px/m, cbSize 1466574, bits offset 54
plan:             ASCII text
program.deb:      Debian binary package (format 2.0), with control.tar.gz , data compression xz
tftp.pcapng:      pcapng capture file - version 1.0
12:18:48 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 
```
- Extracting from these again, we get the final `control` and various other files as well as a `usr` directory.
```bash
12:20:51 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → tar -x -f control.tar
12:20:59 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → tar -x -f data.tar
12:21:04 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → file * 
control:          ASCII text
control.tar:      POSIX tar archive (GNU)
data.tar:         POSIX tar archive (GNU)
debian-binary:    ASCII text
instructions.txt: ASCII text
md5sums:          ASCII text
picture1.bmp:     PC bitmap, Windows 3.x format, 605 x 454 x 24, image size 824464, resolution 5669 x 5669 px/m, cbSize 824518, bits offset 54
picture2.bmp:     PC bitmap, Windows 3.x format, 4032 x 3024 x 24, image size 36578304, resolution 5669 x 5669 px/m, cbSize 36578358, bits offset 54
picture3.bmp:     PC bitmap, Windows 3.x format, 807 x 605 x 24, image size 1466520, resolution 5669 x 5669 px/m, cbSize 1466574, bits offset 54
plan:             ASCII text
program.deb:      Debian binary package (format 2.0), with control.tar.gz , data compression xz
tftp.pcapng:      pcapng capture file - version 1.0
usr:              directory
12:21:07 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 
```
- Analyzing these extracted files one by one, I learned that something called as `steghide` was used. I found this clue in the first file I analyzed, i.e., `control`
```bash
12:21:07 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → cat control 
Package: steghide
Source: steghide (0.5.1-9.1)
Version: 0.5.1-9.1+b1
Architecture: amd64
Maintainer: Ola Lundqvist <opal@debian.org>
Installed-Size: 426
Depends: libc6 (>= 2.2.5), libgcc1 (>= 1:4.1.1), libjpeg62-turbo (>= 1:1.3.1), libmcrypt4, libmhash2, libstdc++6 (>= 4.9), zlib1g (>= 1:1.1.4)
Section: misc
Priority: optional
Description: A steganography hiding tool
 Steghide is steganography program which hides bits of a data file
 in some of the least significant bits of another file in such a way
 that the existence of the data file is not visible and cannot be proven.
 .
 Steghide is designed to be portable and configurable and features hiding
 data in bmp, wav and au files, blowfish encryption, MD5 hashing of
 passphrases to blowfish keys, and pseudo-random distribution of hidden bits
 in the container data.
12:22:36 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 

```
- Hence I downloaded `steghide` on my system, and knew that steganography was involved to hide the password in the images.
- Running the `steghide` on the images, I learned that we need to have a passphrase as well. This was very frustrating as I had to go back and start from `instructions` and `plan` again and read them carefully. Luckily, I found the password in `plan` only as there was a phrase `WITH-DUEDILIGENCE`. Here I understood that `DUEDILIGENCE` could be a special phrase, most likely a password. Using this, I got the flag.
```bash
12:30:01 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → file * 
control:          ASCII text
control.tar:      POSIX tar archive (GNU)
data.tar:         POSIX tar archive (GNU)
debian-binary:    ASCII text
instructions.txt: ASCII text
md5sums:          ASCII text
picture1.bmp:     PC bitmap, Windows 3.x format, 605 x 454 x 24, image size 824464, resolution 5669 x 5669 px/m, cbSize 824518, bits offset 54
picture2.bmp:     PC bitmap, Windows 3.x format, 4032 x 3024 x 24, image size 36578304, resolution 5669 x 5669 px/m, cbSize 36578358, bits offset 54
picture3.bmp:     PC bitmap, Windows 3.x format, 807 x 605 x 24, image size 1466520, resolution 5669 x 5669 px/m, cbSize 1466574, bits offset 54
plan:             ASCII text
program.deb:      Debian binary package (format 2.0), with control.tar.gz , data compression xz
tftp.pcapng:      pcapng capture file - version 1.0
usr:              directory
12:30:03 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → steghide extract -sf picture1.bmp -p DUEDILIGENCE
steghide: could not extract any data with that passphrase!
12:30:31 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → steghide extract -sf picture2.bmp -p DUEDILIGENCE
steghide: could not extract any data with that passphrase!
12:30:38 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → steghide extract -sf picture3.bmp -p DUEDILIGENCE
wrote extracted data to "flag.txt".
12:30:41 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → cat flag.txt
picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}
12:30:46 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads
```
- This was the biggest single challenge that I solved. And seeing the flag say 'hidden in plain sight' made me laugh and wanna break the computer.

## Flag
`picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}`

## Notes
- the tangent I went on was converting the file to hexdump or binary using `xxd` command and then trying to use `binwalk` to extract the data. That was something I thought of, but it didn't work, so I came back after a break of 30 mins and started again from the top, this reminded me that I missed `program.deb` file completely.


## Conceptslearned 
#### Revised Wireshark concepts
This challenge enabled me to revise the concepts of wireshark that I had learned. Revising protocol hierarchy and exporting ojects was fun as well.

#### Learned about Debian Packages
From this challenge, I learned that Debian Packages are also a type of storage, like directories or like archives, and we can extract data from that as well.

#### Revised various compression formats
This challenge helped me revise the concepts of extraction from compressed data as well. I also learned that `ar` is used for Debian Packacges to extract data.

#### Steganography
I cannot really say this was a revision as it was me doing steganography on my own, without the help of AI, which was a new learning experience. Plus I got to know about the existence of `steghide` tool.

## References
- https://play.picoctf.org/practice?category=4&page=1&search=Trivial
- https://cyberchef.io/
- General Google searches which are included as pictures.
----------------------------------------------------------------------------------------------------------------------------------
# tunn3l_v1s10n
In this challenge, we had to first figure out what data we have, then figure out how to reconstruct the data, then resize it to get the final answer.

## Solution 
To solve this challenge, I followed the steps as listed below: 
- I first ran the `file` command on the downloaded data to figure out what data format we have, however it returned just data as result to us. 
```bash
19:14:32 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → file tunn*
tunn3l_v1s10n: data
19:14:36 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 
```
- I did go on a tangent after reading the hint, as mentioned in the notes. But I realised it was not a very fruitful one. I had to go back to analyze the data again and try to find some hint as to what it was. So I ran the `cat` command. This gave me the result as random binary data. So I ran the `xxd` command to convert the binary to a hexdump. After converting it to the hexdump, I also got the idea of magic bytes/headers from a previous challenge that I had solved.
```bash
19:17:55 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → xxd tunn* | more
00000000: 424d 8e26 2c00 0000 0000 bad0 0000 bad0  BM.&,...........
00000010: 0000 6e04 0000 3201 0000 0100 1800 0000  ..n...2.........
00000020: 0000 5826 2c00 2516 0000 2516 0000 0000  ..X&,.%...%.....
00000030: 0000 0000 0000 231a 1727 1e1b 2920 1d2a  ......#..'..) .*
00000040: 211e 261d 1a31 2825 352c 2933 2a27 382f  !.&..1(%5,)3*'8/
00000050: 2c2f 2623 332a 262d 2420 3b32 2e32 2925  ,/&#3*&-$ ;2.2)%
00000060: 3027 2333 2a26 382c 2836 2b27 392d 2b2f  0'#3*&8,(6+'9-+/
00000070: 2623 1d12 0e23 1711 2916 0e55 3d31 9776  &#...#..)..U=1.v
00000080: 668b 6652 996d 569e 7058 9e6f 549c 6f54  f.fR.mV.pX.oT.oT
00000090: ab7e 63ba 8c6d bd8a 69c8 9771 c193 71c1  .~c..m..i..q..q.
000000a0: 9774 c194 73c0 9372 c08f 6fbd 8e6e ba8d  .t..s..r..o..n..
000000b0: 6bb7 8d6a b085 64a0 7455 a377 5a98 6f56  k..j..d.tU.wZ.oV
000000c0: 7652 3a71 523d 6c4f 406d 5244 6e53 4977  vR:qR=lO@mRDnSIw
000000d0: 5e54 5339 3370 5852 7661 5973 5f54 7e6b  ^TS93pXRvaYs_T~k
000000e0: 5e86 7463 7e6a 5976 6250 765e 4c7a 6250  ^.tc~jYvbPv^LzbP
000000f0: 876d 5d83 6959 8d73 639b 8171 9e84 7498  .m].iY.sc..q..t.
00000100: 7e6e 9b81 718d 7363 735a 4a70 5747 5a41  ~n..q.scsZJpWGZA
00000110: 314f 3626 4e37 274f 3828 4f38 2851 3a2a  1O6&N7'O8(O8(Q:*
00000120: 5039 294f 3829 4b35 2950 3a2f 4b35 2a3f  P9)O8)K5)P:/K5*?
00000130: 291e 422e 234b 372c 4531 263f 2b20 432f  ).B.#K7,E1&?+ C/
00000140: 2443 2f24 402a 1f48 3227 4b32 2847 2e24  $C/$@*.H2'K2(G.$
00000150: 4027 1d45 2c22 4c34 284c 3428 4b33 274a  @'.E,"L4(L4(K3'J
00000160: 3226 4c32 244e 3426 5035 2752 3729 5336  2&L2$N4&P5'R7)S6
00000170: 2855 382a 4b30 225d 4234 6349 3949 2f1f  (U8*K0"]B4cI9I/.
00000180: 442b 1b4d 3424 4d36 274a 3324 462c 2048  D+.M4$M6'J3$F, H
00000190: 2e22 462e 2244 2e22 3c26 1b32 2015 301f  ."F."D."<&.2 .0.
000001a0: 1632 231a 3627 1e3c 2b22 3e2b 2442 2c26  .2#.6'.<+">+$B,&
000001b0: 5e44 3e66 4c46 361d 1933 1f1a 3f30 2d13  ^D>fLF6..3..?0-.
000001c0: 0a06 0906 0205 0400 0a05 040b 0605 0d05  ................
000001d0: 050b 0605 0a06 0508 0605 0806 050a 0605  ................
000001e0: 0d05 050d 0505 0b03 0406 0001 0500 0009  ................
000001f0: 0403 0b06 030a 0502 0805 0109 0602 0906  ................
00000200: 0207 0400 0704 0008 0501 0804 0306 0201  ................
19:18:16 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 

```
- Now in this data, I had no clue what the magic bytes meant, I vaguely remembed that starting with `89...` was a PNG image, so I put the first 4 bytes into google search.
![image of google search of 4242 8e26]
- This search actually showed the first link as a website which had a walkthrough of the challenge, so I decided to not visit that site and look up the second link, which was a type of a git repo.
- On visiting the git repo, I learned that `424d` denotes a bmp image. So working on this principle I tried reading through the next bytes. I realised on reading the 11-12 byte that it literally spelled `bad0`. I got suspicious that it was not supposed to be this way. So I googled up how headers of a bmp are supposed to look like. Luckily I found a website which explained the headers very well.
- I realised that these `bad0` headers had to be changed to `3e00` and `2800`. To do this, I first created a file which had only the hex data, nothing else, as shown before:
```bash
19:28:05 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → xxd tunn* | cut -d " " -f 2-9 > hexdumpTunnel
19:29:24 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → nano hexdumpTunnel
19:29:39 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 
```
- On accessing the `hexdumpTunnel` using the nano editor, I changed the bytes as mentioned.
- Then on removing the extra spaces and newline characters present in the file, I created a file. Then reversing that hexdump to get the output of binary to a file, I got an image.
```bash
19:29:39 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → nano hexdumpTunnel
19:31:31 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → cat hexdumpTunnel | tr -d " " | tr -d "\n" > correct_hex
19:32:14 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → xxd -r -p correct_hex > image
```
- Opening the image I got the image as shown
![image of notaflag{sorry}]()
- This was both relieving as well as annoying. So I re-read the name of the challenge, seeing that whenever I get stuck, the name provides a hint.
- To tunnel vision, along with the weird sizing of the image told me that I had to resize the image. I assumed that it was incomplete along the height as having height less that width just felt wrong, so I copied the bytes of the width of image and used it to set the height in the hexdump.
- This gave me a result like this:
![]
- As we can see, I got the flag at the top of the image.

## Flag

`picoCTF{qu1t3_a_v13w_2020}`

## Notes
In this when I first assumed the hint given as 'Weird that it wont display right...' I assumed we needed to find only the strings in this case, so I used the `strings` command to find human readable characters and then started removing all the extra symbols, because I assumed it also had a ROT type cipher as I had solved it in the previous challenge. I realised it wasnt the right path because even strings were very large in number, i.e., 65536+. Now I know not to have preconceived notions as it damages more than anything else. 

## What I learned 

#### Revision of magic bytes
I knew about the existence of magic bytes because of a previous challenge, so solving this challenge reitrated the importance of the Header Bytes, especially in digital forensics.

#### Header Manipulation 
This is a concept that I had an insight into, but was ever comfortable with trying as I always thought I would corrupt the files, but doing and solving this challenge by changing these bytes in the text editor helped me gain confidence that I can manipulate data directly at hex level. 

## References 
- https://play.picoctf.org/practice/challenge/112?category=4&page=1&search=tunn
- https://www.donwalizerjr.com/understanding-bmp/
- https://gist.github.com/leommoore/f9e57ba2aa4bf197ebc5
----------------------------------------------------------------------------------------------------------------------------------
# m00nwalk 
To solve this challenge, we needed to figure out in what format the audio message is being transmitted as to get the photo. 

## Solution 
This was one of the easier challenges in the JTP-2, as this had straight hints. To solve this I followed the steps as listed below: 
- I first downloaded the audio file to hear it once, and on hearing it I realised that I had no clue in what format this was. Hence I went back to the challenge and read the hint to learn that this is actually the same format that Apollo 11 astronauts used to send back images. This was quite fascinating to learn as I never knew about this and I was amazed to learn that we could send images in the audio format.
- Googling and reading up on Quora, I got to know that this was the audio format of SSTV, i.e., Slow-Scan Television. Then I went online and googled SSTV decoders to find the decoder. Uploading the file to the decoder, I got the image which had the flag.
![image of online sstv decoder](/images/forensics/Screenshot-2025-10-24-18-20-32.png)

## Flag
`picoCTF{beep_boop_im_in_space}` 

## Concepts learned

#### Existence of SSTV
I learned about the existence of something know as SSTV and I was very fascinated to learn about this. It really shows that when faced with problems, humans always find a way to get the job done. 

## References
- https://play.picoctf.org/practice?category=4&page=1&search=m00n
- https://www.quora.com/How-were-cameras-in-space-for-moon-landing-broadcasts-sent-back-to-Earth
- https://en.wikipedia.org/wiki/Slow-scan_television
- https://sstv-decoder.mathieurenaud.fr/
----------------------------------------------------------------------------------------------------------------------------------
