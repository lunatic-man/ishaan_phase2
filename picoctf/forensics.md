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
