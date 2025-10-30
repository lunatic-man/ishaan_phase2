# IQ Test
This was a simple Logic Gates test that we had to solve based on the number we got. 

## Solution 
To solve this challenge, I followed the steps as listed below: 
- I first read the description of the challenge to understand what I had to do. After understanding, I converted the number given to us in decimal to binary and then added it as the input to the logic gates.
- I did it wrong the first time as I assumed that we had to take the rightmost bit as x0, but on doing it the second time around, I got the correct answer.
- The process is shown in the image below:
![Logic Gates Solved](/images/hardware/iqtest.png)

## Flag

`nite{100010011000}`


## What I learned
This was a pretty simple challenge with just 12th knowledge requirements. It needed nothing much other than some revision and patience.

## References
- https://drive.google.com/drive/folders/1DG7TRZDwHl-X6s4s8ApypMOOYGNNsUJ1?usp=drive_link
---------------------------------------------------------------------------------------------------------------------------------------
# I Like Logic 
To solve this challenge, we had to analyze a file with `.sal` extension and use Logic 2 Software to get the result. 

## Solution 
To solve this challenge, I followed the steps as followed below: 
- I opened the link present in the Briefing, and I found a `challenge.sal` file. I downloaded it and then googled what is a `.sal` extension. Turns out it stands for Saleae Logic 2 capture file.
- So I downloaded the Logic 2 Software and uploaded the file in this. Using the analyzers, I went about manually checking each analyser filter. Lucking I found the right filter on the second try itself, which was the `Async Serial` Analyzer.
- Reading the terminal output, I found a story with some random data of the form `FCSC{...}`. Confirming with mentor, I got this as the correct flag.
![screenshot of Logic 2 Software](/images/hardware/Screenshot-2025-10-30-16-48-49.png))

## Flag
`FCSC{b1dee4eeadf6c4e60aeb142b0b486344e64b12b40d1046de95c89ba5e23a9925}` 

## What I learned 
#### Logic 2 Software
From this challenge, I learned about the existence of Logic 2 Software and how we can utilize it to analyze files with `.sal` extension.


## References
- https://drive.google.com/drive/folders/1ZEhUzJgVHVTMIlBWrPIHCzH4TnipQPNX
- Google Search on `.sal`
------------------------------------------------------------------------------------------------------------------------------------
# Bare Metal Alchemist
In this challenge we had analyze the ELF file given to us and find the flag hidden in it. 

## Solution 
To solve this challenge, I followed the steps as given below:
- I first put this file in IDA, and it gave me an error that file with `avr` architectures cannot be analyzed by IDA.
- I went online to google what is AVR structure and learned that this is a Harvad Type Architecture where the program and data are stored in seperate physical memory systems.
- I learned this but wasn't sure about how it mattered till I solved the challenge. Then I realised why it mattered.
- So because IDA did not work and I would not use the Pro version, I installed Ghidra. Ghidra took a while to get familiar to and understand how the UI interacted. After this I uploaded the file, `firmware.elf` to analyze the program.
- Now there was a lot of data in it, but in the C code, I managed to find the `main` function. This was a complicated function which took time to understand.
- After understanding this function, I came to the following conclusions.
    1. The program was manipulating the lower register in the  `R25R24` register pair.
    2. This program then entered a subprogram where the values stored at `0x68` was being XORed with `0xA5`. The XOR program is as shown below:
```bash
      R15R14 = 0x68;
      R16 = 0;
      while( true ) {
        Z = (byte *)R15R14;
        R25R24._0_1_ = *(byte *)(uint3)R15R14;
        if ((byte)R25R24 == 0) break;
        Z._1_1_ = (undefined1)(R15R14 >> 8);
        Z._0_1_ = (byte)R25R24 ^ R11;
        if ((byte)R25R24 == 0xa5) break;
        R25R24._0_1_ = DAT_mem_0029;
        R25R24._0_1_ = (byte)R25R24 ^ (byte)R25R24 * '\x02';
        if (((byte)R25R24 & 4) == 0) {
          auStack_7 = (undefined1  [3])0x131;
          z1();
          break;
```
- From this we can also see that the offset is `0x68`. Using this I went around the raw bytes to find this line. However it took me time to understand that the debug had three different section, `code`, `mem` & `codebyte`. Now I was trying to find the flag in the `code` and `mem` addresses. These did not give any fruitful result. Coming to `codebyte` I was able to find the hex.
- The raw hex is as below:
`f1 e3 e6 e6 f1 e3 de f1 cd 94 d6 fa 94 d6 fa d6 ca c8 96 fa d6 94 c8 d5 c9 96 fa 91 d7 c1 d0 94 cb ca fa c3 94 d7 c8 d2 91 d7 c0 d8`
- XORing the above data with `0xA5`, we get the data as shown below:
`A5 54 46 43 43 54 46 7B 54 68 31 73 5F 31 73 5F 73 6F 6D 33 5F 73 31 6D 70 6C 33 5F 34 72 64 75 31 6E 6F 5F 66 31 72 6D 77 34 72 65 7D`
- Converting this to ASCII, I got the flag : `TFCCTF{Th1s_1s_som3_s1mpl3_4rdu1no_f1rmw4re}`


## Flag 
`TFCCTF{Th1s_1s_som3_s1mpl3_4rdu1no_f1rmw4re}`

## Notes
This was one of the most difficult challenges that I have ever solved mainly because of how much code was new and how I needed to understand what Ghidra output was. But I managed to solve it after 2-3 days of effort. 

## What I learned
#### Ghidra Usage
I learned how to utilize Ghidra and its functions to analyze a raw file in a much better way as compared to IDA.

#### C code structures
Despite knowing a bit of C language, I had to learn a lot of new stuff to be able to solve this challenge.

## References
- https://en.wikipedia.org/wiki/AVR_microcontrollers
- https://byte.how/posts/what-are-you-telling-me-ghidra/
------------------------------------------------------------------------------------------------------------------------------------
