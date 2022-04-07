# Classic Passwd
This is a writeup from the "Classic Passwd" Room on Tryhackme. The challenge is to reverse engineer a binary to get the flag.

Well, there are multiple ways to solve this - the first that came to mind were Ghidra and GDB. But before, we should run the binary to check what it actually does.

It seems to ask for a username, and if we provide one, simply closes with an authentication error. The assumption here being that it gives us the flag if the username is correct
and we just don't know the correct username. Now then, on to Ghidra

## Ghidra

Ghidra is pretty straightforward. Immediately when opening, we can see the main function, and that it calls two other functions. 
The first one, already suspiciously called "vuln" seems to compare the provided username with a value at a memory address and return if it is correct.
So our assumption that we have to know the correct username seems to be true. 

Now, it looks like the second function should then be responsible for displaying the flag, right? And indeed it is - we can cleary see the line 

`printf("THM{%d%d}",0x638a78,0x2130);`

Se, we need to convert 0x638a78 and 0x2130 to decimal and append them - easy:

`printf "%d%d" 0x638a78 0x2130`

So, there we have our flag. But....what is the username?

## GDB

Well, we can use GDB for that. Open the binary in gdb and disassemble the main function. Again, we can see the 2 functions - vuln and gfl. 
From before, we know vuln is responsible for checking the username and gfl for displaying the flag. We know the flag, so we disassemble vuln. 

We see a call to strcmp which catches our eye because it seems reasonable to assume when comparing the provided username with the correct username, strcmp might be used.

Immediately before, we see two mov commands - those could be our two usernames. So, let's just set a breakpoint before strcmp and see whats in the registers.

``` 
break *0x000055555555525a

run
```

The program runs, we input our username, and our breakpoint halts just before jumping into strcmp. From disassembling the function, we know the addresses
to our two strings should now be stored in rax and rdx. So...

```
print $rax
$1 = 140737488347856

print (char*) 140737488347856
$2 = 0x7fffffffe2d0 "systemfehler"

print $rdx
$3 = 140737488347986

print (char*) 140737488347986
$4 = 0x7fffffffe352 "*************"
```

Well, there you have it - the username as well. Just for curiosity's sake.




