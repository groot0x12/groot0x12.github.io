---
title: "Back To Basics: Return to Ret2Libc"
date: 2021-10-07T00:00:00-00:00
categories:
  - ctf-writeups
tags:
  - ctf
  - pwn
---

Ret2Libc is an attack that I got exposed to briefly earlier in school but never really got around to understanding at a deep level. This concept resurfaced at the recent DeconstruCT.F 2021, hence I decided to do a deep dive on ret2libc attacks, which I have documented below.

![Question Prompt](/assets/images/ret2libc_prompt.png)

They also gave us the source code of the program.

```c
#include<stdio.h>
#include<string.h>

void disarm_dispenser(){
    char password[256];
    FILE *password_file;
    password_file = fopen("password.txt","r");
    fgets(password,sizeof(*password_file),password_file);
    printf("Enter password to disable dispenser:\n");
    char user_input[256];
    gets(user_input);
    int eq = strncmp(user_input,password,256);
    if (eq != 0) {
        printf("Incorrect password\n");
    }
    else {
        printf("Password correct\n");
        printf("Disarming dispenser (or not lol)...\n");
    }

}

int main() {
    disarm_dispenser();    
}
```

The program prompts the user for a password and compares it to a `password.txt` on the machine. However, we can tell from the source code that providing the right password will not give us the flag in any way. Hence, we likely have to use ret2libc to obtain a shell by overflowing the vulnerable `gets()` function. 

We quickly run `checksec` on the binary to see what security measures are put in place. 

![checksec](/assets/images/ret2libc_checksec.png)

We see that NX is disabled which means we could technically execute code on the stack. This opens up options to use the buffer to contain shellcode and rely on a jump instruction to jump back to the start of the shellcode. However, since ret2libc is explicitly mentioned, let's rely on that exploit.

# A Primer on C Binaries and LibC 
When C binaries are compiled, they may contain various LIBC functions like `gets()`, `printf()` etc. However, we note that these functions are not defined in the program itself, so where are they defined and how does a C program know what to do when it runs these instructions?

> Introducing the LIBC Shared Object! 

The LIBC shared object is a library containing all these C functions that is dynamically linked to our binary at run time. But, on different systems with various protections like ASLR and varying number of processes running, the location of this LIBC shared object would be different as well. How then would a C binary be able to know the location of these specific functions?

# A Primer on GOT and PLT
With modern day binary protections such as ASLR and PIE which cause code to be position-independent and loaded into random addresses in memory, how does a C program locate specific LIBC functions?

Binaries rely on two sections within the binary itself: **Procedure Linkage Table (PLT)** and **Global Offset Table (GOT)**. The PLT is a pseudo-reference to the actual LIBC function, and contains a `jmp` instruction to the location of the actual LIBC function which is stored in the GOT.

Let's examine this flow within a binary itself.