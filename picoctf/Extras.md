# Extra Challenges

## Input Injection 1 - Binary Exploitation
To solve this challenge, we had to cause an overflow and cause the system to execute commands that we wanted. 

### Solution 
To solve this challenge, I followed the steps as below:
- I first downloaded the `.elf` file and then spent time analyzing the debugged code. From the code, I learned a few things
    1. The `main()` function was taking the input using `fgets()` so overflow here was not possible.
    2. It was calling a function called `fun()`, which was taking two pointers as input. This was a new thing to understand, but a key as this is what allowed us to overflow.
    3. The `fun()` function had a `system()` call inside it.
- So while connecting to the terminal for the first time and typing in my own name, I could see that the system was executing the comamnd `uname`. From here I figure out that we needed to change this `uname` to something which allows us to take control of the connection.
- The best was of doing this, was running `/bin/sh` which would allow us to open a shell on the system. Now we needed to figure out how we can overwrite the value. On analyzing the function `fun()` I learned that our input was being copied into the variable `local_1c` and that this had a limit of 10 characters, and anything after this would get carried onto `local_12`, which is what is the argument for `system()` function.
- Using this info, I simply typed in 10 characters, with the next charcaters being `/bin/sh`. This changed the output and did not return `Linux` as before.
- I then typed out ls, which gave me a file, `flag.txt`. This was our flag.
```bash
14:19:34 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~  → nc saffron-estate.picoctf.net 58211
What is your name?
Ishaan 
Goodbye, Ishaan !
Linux
^Z  
[2]+  Stopped                 nc saffron-estate.picoctf.net 58211
14:24:20 ishaan-mishra@ishaan-mishra-Lenovo-G505s ~  → nc saffron-estate.picoctf.net 58211
What is your name?
aaaaaaaaaa/bin/sh
Goodbye, aaaaaaaaaa/bin/sh!
ls
flag.txt
cat flag.txt
picoCTF{0v3rfl0w_c0mm4nd_3766e70a}
```

### Flag
`picoCTF{0v3rfl0w_c0mm4nd_3766e70a}` 

### Notes
I did initally assume that we could cause a simple overflow in the input directly, but I realised later on that it was a `fgets()` function. This however blew my mind up, as it was so new yet so simple.


### What I learned 
#### `system()` usage
From this challenge, I learned that we can use `system()` function to pass any commands that we want inside this. If we can figure out a way to overwrite the arguments, we can take full control of system.

#### Revision of `/bin/sh`
I will admit I had forgotten about this and that we could use this command to start a shell on the target system. It was very fun to re-read and revise these concepts.

### References
- https://play.picoctf.org/practice/challenge/525?category=6&page=1
- https://www.geeksforgeeks.org/cpp/system-call-in-c/
--------------------------------------------------------------------------------------------------------------------------------------
