# GDB baby step 1 
To solve this challenge, we need to find the value that the program stores in the `eax` register at the end of `main` function. 

## Solution 1 

To Solve this challenge, I followed the steps as listed below: 
- I had zero clue about what Reverse Engineering is and what it meant, so I sat and watched the videos given in the PDF to understand what to do and how to solve it. 
- In one of those videos, there was a video for reverse engineering walkthrough. I heavily relied on that video to solve the challenge. 
- The first way of solving it was by using the `objdump` command, which I learned from the video itself. Because I got curious, I went to read the manpage of the `objdump` command and learned that it displays the information about one or more object files and that this information is useful for programmers who are working on compilation tools. 
- I used this along with `-d` argument (as shown in video and as I learned from manpage) to get the disassembled code. This code is mainly assembly instructions, where assembly is a low level programming language. 
- Now I got the result as something shown below: 

```bash 
23:28:42 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → man objdump 
23:29:14 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → objdump -d debugger0_a

debugger0_a:     file format elf64-x86-64


Disassembly of section .init:

0000000000001000 <_init>:
    1000:	f3 0f 1e fa          	endbr64
    1004:	48 83 ec 08          	sub    $0x8,%rsp
    1008:	48 8b 05 d9 2f 00 00 	mov    0x2fd9(%rip),%rax        # 3fe8 <__gmon_start__>
    100f:	48 85 c0             	test   %rax,%rax
    1012:	74 02                	je     1016 <_init+0x16>
    1014:	ff d0                	call   *%rax
    1016:	48 83 c4 08          	add    $0x8,%rsp
    101a:	c3                   	ret

Disassembly of section .plt:

0000000000001020 <.plt>:
    1020:	ff 35 a2 2f 00 00    	push   0x2fa2(%rip)        # 3fc8 <_GLOBAL_OFFSET_TABLE_+0x8>
    1026:	f2 ff 25 a3 2f 00 00 	bnd jmp *0x2fa3(%rip)        # 3fd0 <_GLOBAL_OFFSET_TABLE_+0x10>
    102d:	0f 1f 00             	nopl   (%rax)

Disassembly of section .plt.got:

.
.
.
    119e:	41 5d                	pop    %r13
    11a0:	41 5e                	pop    %r14
    11a2:	41 5f                	pop    %r15
    11a4:	c3                   	ret
    11a5:	66 66 2e 0f 1f 84 00 	data16 cs nopw 0x0(%rax,%rax,1)
    11ac:	00 00 00 00 

00000000000011b0 <__libc_csu_fini>:
    11b0:	f3 0f 1e fa          	endbr64
    11b4:	c3                   	ret

Disassembly of section .fini:

00000000000011b8 <_fini>:
    11b8:	f3 0f 1e fa          	endbr64
    11bc:	48 83 ec 08          	sub    $0x8,%rsp
    11c0:	48 83 c4 08          	add    $0x8,%rsp
    11c4:	c3                   	ret
23:29:25 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 

```
- In this output, I could make out absolutely nothing but I could figure out that `sub`, `add`, `ret` were all keywords, and on reading the right most columns I could see a collection of 3 letter after a `%` sign. 
- I then re-read the challenge, along with watching the video ahead to find out that these are actually something called as registers. Registers are basically temporary data storage slots with blazing fast speed. 
- So having understood that, and realizing that we needed to find the data in `eax` register, I crafted a command like this - `objdump -d debugger0_a | grep 'eax'`. 
- What this command does is that it looks for the `eax` keyword in the output to find the lines with this keyword. Lucky for me it turned out to be just one line with this key word, as shown below: 

```bash 
23:29:25 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → objdump -d debugger0_a | grep 'eax'
    1138:	b8 42 63 08 00       	mov    $0x84263,%eax
23:34:37 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 
```
- I already knew that hexadecimal numbers start with `0x` so in the result, I could see data in the format of `0x`. Using an online converter to convert the hex to decimal, I got a number, `549698`. Putting this in, I got the flag.


## Solution 2 
I also found another way of solving the challenge by continuing to watch the entire YouTube video where I learned that we can also use the IDA debugger, decompiler to get the result of the full code. 

- I first installed IDA which turned out to be a pain as I was not able to figure out what is a `.hexlic` file. Ended up wasting 15 mins on just finding out that file. 
- After installing it, I ran the program and it asked me to chose which file to disassemble. Giving it the file that we wanted to disassemble, I figured out the value stored in the `eax` register as shown in the image.

![IDA Photo](/images/revEngg/Screenshot-2025-10-23-00-11-32.png)

- Putting the data into an online hex to binary converter, I got the flag. 

![Hex to Binary](/images/revEngg/Screenshot-2025-10-22-23-57-18.png)

## Flag 
`picoCTF{549698}`

## Concepts Learned 

In this challenge, I learned the following concepts:

#### Existence of Registers and how they are utilized
I learned that registers are smaller storage units as compared to RAM and these allow for lightning fast transfer of data. They are however volatile and do not store data for long.

#### Existence of Assembly Instructions and how low level they are 
I learned that we can use Assembly Instructions to directly deal with the aforementioned registers and how these Instructions can also be used to store and manipulate data.

#### Usage of `objdump` command along with necessary arguments
I learned the usage of `objdump` command along with the needed arguments to ensure we can disassemble the binary that we have been given and convert it to a human readable format.

#### Usage of IDA tool for debugging
I learned how we can use the IDA tool for directly debugging and getting the result of the binary data into human understandable terms so that we can reverse engineer software pretty effectively. 

## Resources
- https://play.picoctf.org/practice?page=1&search=GDB%20baby%201
- https://www.youtube.com/watch?v=gh2RXE9BIN8
- https://my.hex-rays.com/dashboard/download-center/installers/release/9.2/ida-free
- https://www.rapidtables.com/convert/number/hex-to-decimal.html?x=86342
----------------------------------------------------------------------------------------------------------------------------------
# ARMssembly 1 
In this challenge, we needed to find out the process that the program does and then figure out what argument would allow us to win this program. 

## Solution 

To solve this challenge, I followed the steps as listed below: 
- I first read the challenge statement and downloaded the file which was mentioned there. On downloading and using `file` command to figure out what type of file it is, I learned that it was a simple ASCII text.
- Because of it being a simple ASCII text, I used to `cat` command to find the contents of the file only to realise it is anything but simple, and that the assembler source that I saw before ASCII text meant something.

```bash

12:42:00 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → ls
chall_1.S
12:42:00 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → file chall_1.S
chall_1.S: assembler source, ASCII text
12:42:08 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → cat chall_1.S
	.arch armv8-a
	.file	"chall_1.c"
	.text
	.align	2
	.global	func
	.type	func, %function
func:
	sub	sp, sp, #32
	str	w0, [sp, 12]
	mov	w0, 79
	str	w0, [sp, 16]
	mov	w0, 7
	str	w0, [sp, 20]
	mov	w0, 3
	str	w0, [sp, 24]
	ldr	w0, [sp, 20]
	ldr	w1, [sp, 16]
	lsl	w0, w1, w0
	str	w0, [sp, 28]
	ldr	w1, [sp, 28]
	ldr	w0, [sp, 24]
	sdiv	w0, w1, w0
	str	w0, [sp, 28]
	ldr	w1, [sp, 28]
	ldr	w0, [sp, 12]
	sub	w0, w1, w0
	str	w0, [sp, 28]
	ldr	w0, [sp, 28]
	add	sp, sp, 32
	ret
	.size	func, .-func
	.section	.rodata
	.align	3
.LC0:
	.string	"You win!"
	.align	3
.LC1:
	.string	"You Lose :("
	.text
	.align	2
	.global	main
	.type	main, %function
main:
	stp	x29, x30, [sp, -48]!
	add	x29, sp, 0
	str	w0, [x29, 28]
	str	x1, [x29, 16]
	ldr	x0, [x29, 16]
	add	x0, x0, 8
	ldr	x0, [x0]
	bl	atoi
	str	w0, [x29, 44]
	ldr	w0, [x29, 44]
	bl	func
	cmp	w0, 0
	bne	.L4
	adrp	x0, .LC0
	add	x0, x0, :lo12:.LC0
	bl	puts
	b	.L6
.L4:
	adrp	x0, .LC1
	add	x0, x0, :lo12:.LC1
	bl	puts
.L6:
	nop
	ldp	x29, x30, [sp], 48
	ret
	.size	main, .-main
	.ident	"GCC: (Ubuntu/Linaro 7.5.0-3ubuntu1~18.04) 7.5.0"
	.section	.note.GNU-stack,"",@progbits
12:42:14 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  →

```
- Now I figured out that this is assembly code, but I was not able to understand what it means at all. So I went through the videos given in the PDF to understand what it is about, and I was still pretty confused.
- I did learn a few things from the videos, mainly `main` is where our actual code is, `func` is just a function that we define and use and `bl` is how we call a function. I also learned that `atoi` is a function that converts the text we entered as ASCII into Int for calculations.
- Drawing on experiences and lessons from the previous challenge, I knew that we had to learn how to manipulate registers. However I was not able to find the typical `eax` format in this code which I had in the previous code. So I went online and googled about the `w0` , `x1` etc, which I assumed were registers as the videos taught that much.
![Image of Google Search of w0,w1,x0,x1]()
- Now after understanding that these are also registers, I learned what are the commands here and what do they do. Learning the assembly code was one of the toughest things I have ever done, and I will be honest, I am still confused about it.
- I will try my best to explain each line here based on my understanding. First of all, there is something called as registers which are fast storage units, but volatile, that we use to store data. Assembly teaches us how to manipulate the data in these.
- These registers are represented by `w0` or `x0` or `eax`. The first two notations are used to access different parts of register `0`. `w` is used to access the lower 32 bits, whereas `x` is used to access all 64 bits of a register. The `eax` is just another way of accessing the registers, however it points to a specific register, and `e` here denotes 32 bit architecture.
- Now in these registers, we can store and manipulate data. To do this, we use mnemonic commands, like `str`, `ldr`, `add`, `sub`, etc.
- These things together make up an assembly code. Coming back to the code that we got, this code starts at `func` as denoted by the line `.global func`. I do not know what other lines mean.
- Now coming to `main` block as explained in video, we can see that we first use the `stp` command along with `x29` and `x30` registers. This instruction is a Store Pair instruction. It stores two registers to a location in memory. This location in memory is given by `[sp, -48]!`. What this does is it stores `x29` and `x30` registers in the address pointed to by `sp` register, at an offset of -48. 
