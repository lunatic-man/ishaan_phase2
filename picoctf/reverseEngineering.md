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
![Image of Google Search of w0,w1,x0,x1](/images/revEngg/Screenshot-2025-10-23-21-54-45.png)
- Now after understanding that these are also registers, I learned what are the commands here and what do they do. Learning the assembly code was one of the toughest things I have ever done, and I will be honest, I am still confused about it.
- I will try my best to explain each line here based on my understanding. First of all, there is something called as registers which are fast storage units, but volatile, that we use to store data. Assembly teaches us how to manipulate the data in these.
- These registers are represented by `w0` or `x0` or `eax`. The first two notations are used to access different parts of register `0`. `w` is used to access the lower 32 bits, whereas `x` is used to access all 64 bits of a register. The `eax` is just another way of accessing the registers, however it points to a specific register, and `e` here denotes 32 bit architecture.
- Now in these registers, we can store and manipulate data. To do this, we use mnemonic commands, like `str`, `ldr`, `add`, `sub`, etc.
- These things together make up an assembly code. Coming back to the code that we got, this code starts at `func` as denoted by the line `.global func`. I do not know what other lines mean.
- Now coming to `main` block as explained in video, we can see that we first use the `stp` command along with `x29` and `x30` registers. This instruction is a Store Pair instruction. It stores two registers to a location in memory. This location in memory is given by `[sp, -48]!`. To understand this line, I had to go in depth and analyze what actually happens here. I spent about an hour learning about registers, stack, stack frame and all other things. It was quite interesting. So when an OS loads up, it creates space in RAM, called as Stack. This stack is a huge quantity of memory. The top of this Stack is denoted by a register, `sp` (Stack Pointer). This register holds the memory address of the top of the Stack.
- Now when we call a function, we may need to store certain values which are already in register to the RAM as RAM can store it for longer duration and registers may get overwritten during execution. To do this, we save a type of a copy in the Stack. We create space, called as Stack Frame (Stack Frames are specific to a function and the choice of the designer), where we store these registers. To create a Stack Frame we use the command `sub sp, sp, #32`. What this command does is, is that it tells the system to reduce the value stored in the `sp` register, i.e., it 'slides' the pointer downwards to create space between the top of the stack and the stack pointer, which becomes the stack frame for the function. The example quotes slides down the `sp` by 32 bytes, creating space for exactly 32 x 8 bits. 
- With that out of the way and its understanding, we can now move on to what is the actual logic and what is going on in the code. So reading the code and understanding it as it works, we can see that we first take the input and convert it to integer, then we call the `func` function. In this function, we first create a space of 32 bytes as our stack frame. Then in the function we had some instructions that were new to me, googling these functions I understood that these instructions first store the values in various 32 bit registers, and then manipulate these.
- So in `func` going line by line, we see that first `w0` has value as 79 and then we store this register onto the stack frame to at sp + 12.
- Continuing so on we get our stack frame ready. Now here `lsl` and `sdiv` were new instructions which refer to bitwise left shift operator and division operator respectively.
- Following the algorithm, we get an equation which looks like { (79 << 7)/3 - x }. Here x becomes the input that we have gotten from the terminal.
- Now according to challenge, we need to get the win condition. Reading the code and analyzing it further, we see the `cmp` and `bne` instructions. What these two do is that they test whether a condition is true or false, and if it is false, that is `cmp w0, 0` is false, it enters `bne .L4`, that is it calls `.L4` code. However we know that this is not the win condition. Hence we need to ensure `w0` becomes 0, meaning the equation above becomes 0. So this means that we have to get value of x such that above equation is zero. Solving this, I got the value of x as 3370. Putting this decimal in decimal to hex, I got the value as `D2A`. I did try putting the flag as `picoCTF{D2A}` but it failed so I re-read how we need to submit the flag and submitted it in the correct format with padded zeros.


## Flag 

`picoCTF{00000d2a}` 

## Concepts Learned
#### Stack and Stack Frame
From this challenge, I learned about stack and stack frame and how we initialize a stack to create stack frame specific to a function, along with storing the data in the memory. 

#### Assembly Language
I learned how we can use low level language to directly interact with hardware and memory and utilize it to the best of our abilities. 

## Notes
In this challenge, I initially tried uploading the code to the IDA disassembler thinking that it would turn the assembly directly into C or some other program, but it gave an error. I then googled that there is also a Ghidra disassembler that decompiles directly into C type code. So will try that next time.

## References 
- https://play.picoctf.org/practice?category=3&page=1&search=ARMss
- https://youtube.com/watch?v=1d-6Hv1c39c
- https://youtube.com/watch?v=gh2RXE9BIN8
- https://www.geeksforgeeks.org/computer-organization-architecture/what-is-assembly-language/
---------------------------------------------------------------------------------------------------------------------------------
# Vault Door 3 
To solve this challenge, we need to create a password such that it meets the conditions set by the program. 

## Solution 
To solve this challenge, I followed the steps as listed below: 
- I first read through the code to understand what it does. Despite having limited knowledge about js, I could understand that the main function has the code to take the input from the user, and from this input it selects the substring which starts with `picoCTF{`. This is shown as below

```bash

20:35:36 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → ls
VaultDoor3.java
20:35:37 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → file Vault*
VaultDoor3.java: C++ source, ASCII text
20:35:42 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → cat Vault*
import java.util.*;

class VaultDoor3 {
    public static void main(String args[]) {
        VaultDoor3 vaultDoor = new VaultDoor3();
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter vault password: ");
        String userInput = scanner.next();
	String input = userInput.substring("picoCTF{".length(),userInput.length()-1);
	if (vaultDoor.checkPassword(input)) {
	    System.out.println("Access granted.");
	} else {
	    System.out.println("Access denied!");
        }
    }

    // Our security monitoring team has noticed some intrusions on some of the
    // less secure doors. Dr. Evil has asked me specifically to build a stronger
    // vault door to protect his Doomsday plans. I just *know* this door will
    // keep all of those nosy agents out of our business. Mwa ha!
    //
    // -Minion #2671
    public boolean checkPassword(String password) {
        if (password.length() != 32) {
            return false;
        }
        char[] buffer = new char[32];
        int i;
        for (i=0; i<8; i++) {
            buffer[i] = password.charAt(i);
        }
        for (; i<16; i++) {
            buffer[i] = password.charAt(23-i);
        }
        for (; i<32; i+=2) {
            buffer[i] = password.charAt(46-i);
        }
        for (i=31; i>=17; i-=2) {
            buffer[i] = password.charAt(i);
        }
        String s = new String(buffer);
        return s.equals("jU5t_a_sna_3lpm12g94c_u_4_m7ra41");
    }
}
20:35:46 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~/Downloads  → 

```
- Now in this code, I also found the check password function in it. Here I had to go online and google why do we use `.` in the js code.

![screenshot of google search of .](/images/revEngg/Screenshot-2025-10-23-22-03-19.png)
![Screenshot of google search of .](/images/revEngg/Screenshot-2025-10-23-22-03-33.png)

- Learning that `.` is a fundamental operator in js, used for accessing either properties or for accessing the functions.
- Using this I realised that the line ```if (vaultDoor.checkPassword(input)) {``` is just a way of calling the function, I moved on to understand the function.
- In the function I realised that we first take a password that has to be 32 characters long, and then this password has to match the final string, the `s.equals(...)` part to ensure that we get the return value as 1 and hence access granted. I also learned that `charAt()` gives the character of the string at a particular specified index
- I solved this on pen and paper as shown below:
  ![image of pen and paper solve](/images/revEngg/20251023_211314.jpg)
  ![image1 of pen and paper solve](/images/revEngg/20251023_211320.jpg)
- Solving this, I got the flag.

## Flag
`picoCTF{jU5t_a_s1mpl3_an4gr4m_4_u_c79a21}`

## What I learned 
#### JavaScript Behaviour
I learned about how js works and what is the `.` operator in it and how we can use it. 

#### Logic Practice 
The reverse building of the password from the buffer turned out to be a nice test for my mental faculties and allowed me to refresh my brain. 

## References 
- https://play.picoctf.org/practice?category=3&page=1&search=Vault%20Door
- https://www.w3schools.com/java/ref_string_charat.asp
- https://www.w3schools.com/java/java_intro.asp
----------------------------------------------------------------------------------------------------------------------------------
