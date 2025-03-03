# Bomb_Lab解答

## phase_1

### 汇编代码

#### **phase1**

```assembly
0000000000400ef0 <phase_1>:
  400ef0:	48 83 ec 08          	sub    $0x8,%rsp
  
  400ef4:	be c0 24 40 00       	mov    $0x4024c0,%esi #从0x4024c0处获取字符串存入esi
  400ef9:	e8 c0 03 00 00       	callq  4012be <strings_not_equal> #调用函数
  
  400efe:	85 c0                	test   %eax,%eax #检测是否为0
  400f00:	74 05                	je     400f07 <phase_1+0x17> #正确的话跳过炸弹
  400f02:	e8 62 06 00 00       	callq  401569 <explode_bomb>
  
  400f07:	48 83 c4 08          	add    $0x8,%rsp0.
  400f0b:	c3                   	retq   
```

#### **strings_not_equal**

```assembly
00000000004012be <strings_not_equal>:
  4012be:	41 54                	push   %r12
  4012c0:	55                   	push   %rbp
  4012c1:	53                   	push   %rbx
  4012c2:	48 89 fb             	mov    %rdi,%rbx
  4012c5:	48 89 f5             	mov    %rsi,%rbp
  4012c8:	e8 d4 ff ff ff       	callq  4012a1 <string_length>
  4012cd:	41 89 c4             	mov    %eax,%r12d
  4012d0:	48 89 ef             	mov    %rbp,%rdi
  4012d3:	e8 c9 ff ff ff       	callq  4012a1 <string_length>
  4012d8:	ba 01 00 00 00       	mov    $0x1,%edx
  4012dd:	41 39 c4             	cmp    %eax,%r12d
  4012e0:	75 3e                	jne    401320 <strings_not_equal+0x62>
  4012e2:	0f b6 03             	movzbl (%rbx),%eax
  4012e5:	84 c0                	test   %al,%al
  4012e7:	74 24                	je     40130d <strings_not_equal+0x4f>
  4012e9:	3a 45 00             	cmp    0x0(%rbp),%al
  4012ec:	74 09                	je     4012f7 <strings_not_equal+0x39>
  4012ee:	66 90                	xchg   %ax,%ax
  4012f0:	eb 22                	jmp    401314 <strings_not_equal+0x56>
  4012f2:	3a 45 00             	cmp    0x0(%rbp),%al
  4012f5:	75 24                	jne    40131b <strings_not_equal+0x5d>
  4012f7:	48 83 c3 01          	add    $0x1,%rbx
  4012fb:	48 83 c5 01          	add    $0x1,%rbp
  4012ff:	0f b6 03             	movzbl (%rbx),%eax
  401302:	84 c0                	test   %al,%al
  401304:	75 ec                	jne    4012f2 <strings_not_equal+0x34>
  401306:	ba 00 00 00 00       	mov    $0x0,%edx
  40130b:	eb 13                	jmp    401320 <strings_not_equal+0x62>
  40130d:	ba 00 00 00 00       	mov    $0x0,%edx
  401312:	eb 0c                	jmp    401320 <strings_not_equal+0x62>
  401314:	ba 01 00 00 00       	mov    $0x1,%edx
  401319:	eb 05                	jmp    401320 <strings_not_equal+0x62>
  40131b:	ba 01 00 00 00       	mov    $0x1,%edx
  401320:	89 d0                	mov    %edx,%eax
  401322:	5b                   	pop    %rbx
  401323:	5d                   	pop    %rbp
  401324:	41 5c                	pop    %r12
  401326:	c3                   	retq   
```

### 解答过程

要运行经过函数 `strings_not_equal`目标字符串才被加载，才能尝试print目标字符串

```cmd
(gdb) break explode_bomb // 在爆炸处打断点
(gdb) print (char*)0x4024c0 // 获取key
```

答案即为 `I am the mayor. I can do anything I want.`

---

## phase_2

### 汇编代码

#### **phase2**

```assembly
0000000000400f0c <phase_2>:
  400f0c:	55                   	push   %rbp
  400f0d:	53                   	push   %rbx
  400f0e:	48 83 ec 28          	sub    $0x28,%rsp
  400f12:	48 89 e6             	mov    %rsp,%rsi
  
  400f15:	e8 85 06 00 00       	callq  40159f <read_six_numbers> #调用函数读入数据
  400f1a:	83 3c 24 01          	cmpl   $0x1,(%rsp) #比较输入的第一个数与1
  400f1e:	74 20                	je     400f40 <phase_2+0x34> #相等则跳转 
  400f20:	e8 44 06 00 00       	callq  401569 <explode_bomb> #不相等爆炸
  400f25:	eb 19                	jmp    400f40 <phase_2+0x34>
  
  400f27:	8b 43 fc             	mov    -0x4(%rbx),%eax #读入一个数存入eax
  400f2a:	01 c0                	add    %eax,%eax #把这个数加上自己（x *= 2）
  400f2c:	39 03                	cmp    %eax,(%rbx) #与下一个数比较
  400f2e:	74 05                	je     400f35 <phase_2+0x29> #相等则跳转
  400f30:	e8 34 06 00 00       	callq  401569 <explode_bomb> #不相等爆炸
  400f35:	48 83 c3 04          	add    $0x4,%rbx #每次移动4位，逐个读入数字
  400f39:	48 39 eb             	cmp    %rbp,%rbx #比较，确定循环次数
  400f3c:	75 e9                	jne    400f27 <phase_2+0x1b>
  400f3e:	eb 0c                	jmp    400f4c <phase_2+0x40>
  
  400f40:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx
  400f45:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp
  400f4a:	eb db                	jmp    400f27 <phase_2+0x1b> 
  400f4c:	48 83 c4 28          	add    $0x28,%rsp
  400f50:	5b                   	pop    %rbx
  400f51:	5d                   	pop    %rbp
  400f52:	c3                   	retq   
```

#### **read_six_numbers**

```assembly
000000000040159f <read_six_numbers>:
  40159f:	48 83 ec 18          	sub    $0x18,%rsp
  4015a3:	48 89 f2             	mov    %rsi,%rdx
  4015a6:	48 8d 4e 04          	lea    0x4(%rsi),%rcx
  4015aa:	48 8d 46 14          	lea    0x14(%rsi),%rax
  4015ae:	48 89 44 24 08       	mov    %rax,0x8(%rsp)
  4015b3:	48 8d 46 10          	lea    0x10(%rsi),%rax
  4015b7:	48 89 04 24          	mov    %rax,(%rsp)
  4015bb:	4c 8d 4e 0c          	lea    0xc(%rsi),%r9
  4015bf:	4c 8d 46 08          	lea    0x8(%rsi),%r8
  4015c3:	be b5 27 40 00       	mov    $0x4027b5,%esi
  4015c8:	b8 00 00 00 00       	mov    $0x0,%eax
  4015cd:	e8 5e f6 ff ff       	callq  400c30 <__isoc99_sscanf@plt>
  4015d2:	83 f8 05             	cmp    $0x5,%eax
  4015d5:	7f 05                	jg     4015dc <read_six_numbers+0x3d>
  4015d7:	e8 8d ff ff ff       	callq  401569 <explode_bomb>
  4015dc:	48 83 c4 18          	add    $0x18,%rsp
  4015e0:	c3                   	retq   
```

### 解答过程

根据上述代码分析我们可知：我们应输入六个数字，使其能够满足以下条件：

> step1:输入的第一个数应为1
>
> step2:后一个数应为前一个的两倍

由此易得答案为`1 2 4 8 16 32`

---

## phase_3

### 汇编代码

#### phase3

```assembly
0000000000400f53 <phase_3>:
  400f53:	48 83 ec 18          	sub    $0x18,%rsp # 分配 0x18 bytes 的栈空间
  400f57:	48 8d 4c 24 08       	lea    0x8(%rsp),%rcx # 将 rcx 指向栈上的第 8 个字节
  400f5c:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx # 将 rdx 指向栈上的第 12 个字节
  400f61:	be c1 27 40 00       	mov    $0x4027c1,%esi # 将 0x4027c1 存入 esi 寄存器
  400f66:	b8 00 00 00 00       	mov    $0x0,%eax # 将 0 存入 eax 寄存器
  400f6b:	e8 c0 fc ff ff       	callq  400c30 <__isoc99_sscanf@plt>  # 调用 sscanf 函数读取两个输入值
  400f70:	83 f8 01             	cmp    $0x1,%eax # 检查 sscanf 的返回值是否大于 1
  400f73:	7f 05                	jg     400f7a <phase_3+0x27> # 如果大于 1,跳转到 400f7a
  400f75:	e8 ef 05 00 00       	callq  401569 <explode_bomb> # 否则,调用 explode_bomb 函数

  400f7a:	83 7c 24 0c 07       	cmpl   $0x7,0xc(%rsp) # 检查第一个输入值是否大于 7
  400f7f:	77 3c                	ja     400fbd <phase_3+0x6a> # 如果大于 7,跳转到 400fbd
  400f81:	8b 44 24 0c          	mov    0xc(%rsp),%eax # 将第一个输入值存入 eax 寄存器
  400f85:	ff 24 c5 20 25 40 00 	jmpq   *0x402520(,%rax,8) # 使用第一个输入值作为索引跳转到跳转表中对应的位置

  # 以下为跳转表中的各个分支
  400f8c:	b8 9a 01 00 00       	mov    $0x19a,%eax # 将 0x19a 存入 eax 寄存器
  400f91:	eb 3b                	jmp    400fce <phase_3+0x7b> # 跳转到 400fce
  400f93:	b8 04 02 00 00       	mov    $0x204,%eax # 将 0x204 存入 eax 寄存器
  400f98:	eb 34                	jmp    400fce <phase_3+0x7b> # 跳转到 400fce
  # 其他分支类似
  400f9a:	b8 4d 03 00 00       	mov    $0x34d,%eax
  400f9f:	eb 2d                	jmp    400fce <phase_3+0x7b>
  400fa1:	b8 f5 01 00 00       	mov    $0x1f5,%eax
  400fa6:	eb 26                	jmp    400fce <phase_3+0x7b>
  400fa8:	b8 ee 02 00 00       	mov    $0x2ee,%eax
  400fad:	eb 1f                	jmp    400fce <phase_3+0x7b>
  400faf:	b8 95 02 00 00       	mov    $0x295,%eax
  400fb4:	eb 18                	jmp    400fce <phase_3+0x7b>
  400fb6:	b8 00 02 00 00       	mov    $0x200,%eax
  400fbb:	eb 11                	jmp    400fce <phase_3+0x7b>
  
  400fbd:	e8 a7 05 00 00       	callq  401569 <explode_bomb> # 若上述未跳转成功，会爆炸
  400fc2:	b8 00 00 00 00       	mov    $0x0,%eax # 将 0x0 存入 eax 寄存器
  400fc7:	eb 05                	jmp    400fce <phase_3+0x7b> # 跳转到 400fce
  400fc9:	b8 4e 02 00 00       	mov    $0x24e,%eax # 将 0x24e 存入 eax 寄存器
  
  400fce:	3b 44 24 08          	cmp    0x8(%rsp),%eax # 比较 eax 和第二个输入值
  400fd2:	74 05                	je     400fd9 <phase_3+0x86> # 如果相等,跳转到 400fd9
  400fd4:	e8 90 05 00 00       	callq  401569 <explode_bomb> # 否则,调用 explode_bomb 函数
  400fd9:	48 83 c4 18          	add    $0x18,%rsp # 释放栈空间
  400fdd:	c3                   	retq   # 返回
```

### 解答过程

1. 首先确定输入的格式

```cmd
(gdb) x /s 0x4027c1
0x4027c1: "%d %d"
```

可知应该输入两个整数

2. 接下来分析跳转表

```cmd
(gdb) x /8a 0x402520
0x402520:	0x400f8c <phase_3+57>	0x400fc9 <phase_3+118>
0x402530:	0x400f93 <phase_3+64>	0x400f9a <phase_3+71>
0x402540:	0x400fa1 <phase_3+78>	0x400fa8 <phase_3+85>
0x402550:	0x400faf <phase_3+92>	0x400fb6 <phase_3+99>
```

3. 综合分析跳转表和各分支内容，可得到

一组合理的答案为`2 516`

---

## phase_4

### 汇编代码

#### **phase4**

```assembly
0000000000401016 <phase_4>:
  401016:	48 83 ec 18          	sub    $0x18,%rsp
  40101a:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  40101f:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  
  401024:	be c1 27 40 00       	mov    $0x4027c1,%esi #读入两个整数
  
  401029:	b8 00 00 00 00       	mov    $0x0,%eax #0存在eax
  40102e:	e8 fd fb ff ff       	callq  400c30 <__isoc99_sscanf@plt> #读取的数值的个数
  401033:	83 f8 02             	cmp    $0x2,%eax #是否为2
  401036:	75 0c                	jne    401044 <phase_4+0x2e> #不是则炸弹爆炸
  
  401038:	8b 44 24 0c          	mov    0xc(%rsp),%eax #第一个数存在eax
  40103c:	83 e8 02             	sub    $0x2,%eax #第一个数-2
  40103f:	83 f8 02             	cmp    $0x2,%eax #比较当前eax与2
  401042:	76 05                	jbe    401049 <phase_4+0x33> # <=2的时候跳到401049
  401044:	e8 20 05 00 00       	callq  401569 <explode_bomb> #否则爆炸
  
  401049:	8b 74 24 0c          	mov    0xc(%rsp),%esi #第一个数放在esi
  40104d:	bf 05 00 00 00       	mov    $0x5,%edi  #把5放在edi里
  401052:	e8 87 ff ff ff       	callq  400fde <func4> #调用fun4
  401057:	3b 44 24 08          	cmp    0x8(%rsp),%eax #比较第二个数和eax
  40105b:	74 05                	je     401062 <phase_4+0x4c> #相等跳转到401062
  40105d:	e8 07 05 00 00       	callq  401569 <explode_bomb> #不相等爆炸
  
  401062:	48 83 c4 18          	add    $0x18,%rsp
  401066:	c3                   	retq   
```

#### func4

```assembly
0000000000400fde <func4>:  #esi是第一个数，edi是5
  400fde:	41 54                	push   %r12
  400fe0:	55                   	push   %rbp
  400fe1:	53                   	push   %rbx
 
  400fe2:	89 fb                	mov    %edi,%ebx #edi放在edx中
  400fe4:	85 ff                	test   %edi,%edi #比较edi 
  400fe6:	7e 24                	jle    40100c <func4+0x2e> #edi<=0跳转到40100c
  400fe8:	89 f5                	mov    %esi,%ebp #esi放在ebp
  400fea:	89 f0                	mov    %esi,%eax #esi放在eax
  
  400fec:	83 ff 01             	cmp    $0x1,%edi #比较edi和1
  400fef:	74 20                	je     401011 <func4+0x33> #若相等跳转到401011
  400ff1:	8d 7f ff             	lea    -0x1(%rdi),%edi #不相等则将edi-1存入edi
  
  400ff4:	e8 e5 ff ff ff       	callq  400fde <func4> #递归调用func4
  400ff9:	44 8d 24 28          	lea    (%rax,%rbp,1),%r12d #eax+ebp存在r12d
  400ffd:	8d 7b fe             	lea    -0x2(%rbx),%edi #rbx-2存入edi
  
  401000:	89 ee                	mov    %ebp,%esi #ebp移动到esi
  401002:	e8 d7 ff ff ff       	callq  400fde <func4> #递归调用func4
  401007:	44 01 e0             	add    %r12d,%eax #r12d+eax存入eax
  40100a:	eb 05                	jmp    401011 <func4+0x33> #跳转到401011
  
  40100c:	b8 00 00 00 00       	mov    $0x0,%eax #0放在eax
  
  401011:	5b                   	pop    %rbx
  401012:	5d                   	pop    %rbp
  401013:	41 5c                	pop    %r12
  401015:	c3                   	retq   
```

### 解答过程

`phase_4`中向`func4`传入两个参数，要求：

**x = func4(5,y)**

可将`func4`函数翻译为如下代码

```c
int func4(int x, int y) {
    if (x == 0) 
        return 0;
    if (x == 1) 
        return y;
    // 注意递归调用规则
    return func4(x - 1, y) + func4(x - 2, y) + y;
}
```

由此可以推理得到答案为`24 2`

---

## phase_5

### 汇编代码

#### **phase5**

```assembly
0000000000401067 <phase_5>:
  401067:	53                   	push   %rbx
  401068:	48 89 fb             	mov    %rdi,%rbx #输入字符串存入rbx
  
  40106b:	e8 31 02 00 00       	callq  4012a1 <string_length> #调用函数，字符串长度存入eax
  401070:	83 f8 06             	cmp    $0x6,%eax #比较eax与6
  401073:	74 05                	je     40107a <phase_5+0x13> #相等则跳转到40107a
  401075:	e8 ef 04 00 00       	callq  401569 <explode_bomb> #不相等则爆炸
  
  40107a:	b8 00 00 00 00       	mov    $0x0,%eax #初始化eax=0
  40107f:	ba 00 00 00 00       	mov    $0x0,%edx #初始化edx=0
  
  401084:	0f b6 0c 03          	movzbl (%rbx,%rax,1),%ecx #逐个访问字符串中的字符
  401088:	83 e1 0f             	and    $0xf,%ecx #获取低4位
  40108b:	03 14 8d 60 25 40 00 	add    0x402560(,%rcx,4),%edx #根据低4位从数组中取值加入edx
  
  401092:	48 83 c0 01          	add    $0x1,%rax #rax+1
  401096:	48 83 f8 06          	cmp    $0x6,%rax #rax与6比较
  40109a:	75 e8                	jne    401084 <phase_5+0x1d> #不相等则跳转回401084
  
  40109c:	83 fa 31             	cmp    $0x31,%edx #edx与49比较
  40109f:	74 05                	je     4010a6 <phase_5+0x3f> #相等跳转到4010a6
  4010a1:	e8 c3 04 00 00       	callq  401569 <explode_bomb> #不相等爆炸
  4010a6:	5b                   	pop    %rbx
  4010a7:	c3                   	retq   
```

#### **string_length**

```assembly
00000000004012a1 <string_length>:
  4012a1:	80 3f 00             	cmpb   $0x0,(%rdi)
  4012a4:	74 12                	je     4012b8 <string_length+0x17>
  4012a6:	48 89 fa             	mov    %rdi,%rdx
  4012a9:	48 83 c2 01          	add    $0x1,%rdx
  4012ad:	89 d0                	mov    %edx,%eax
  4012af:	29 f8                	sub    %edi,%eax
  4012b1:	80 3a 00             	cmpb   $0x0,(%rdx)
  4012b4:	75 f3                	jne    4012a9 <string_length+0x8>
  4012b6:	f3 c3                	repz retq 
  4012b8:	b8 00 00 00 00       	mov    $0x0,%eax
  4012bd:	c3                   	retq   
```

### 解答过程

首先应获取对应数组

```cmd
(gdb) x/30dw 0x402560
0x402560 <array.3159>:	2	10	6	1
0x402570 <array.3159+16>:	12	16	9	3
0x402580 <array.3159+32>:	4	7	14	5
0x402590 <array.3159+48>:	11	8	15	13
```

之后可通过各字符ASCII码的低四位为索引，获得一个对应表

| o    | a,p  | b,q  | c,r  |
| ---- | ---- | ---- | ---- |
| 2    | 10   | 6    | 1    |
| d,s  | e,t  | f,u  | g,v  |
| 12   | 16   | 9    | 3    |
| h,w  | i,x  | j,y  | k,z  |
| 4    | 7    | 14   | 5    |
| l    | m    | n    | o    |
| 11   | 8    | 15   | 13   |

为使得输入的字符对应数据求和=49

一组合理的答案为`abgljk`

---

## phase 6

### 汇编代码

```assembly
00000000004010a8 <phase_6>:
  4010a8:	41 55                	push   %r13 # 保存 r13 寄存器的值
  4010aa:	41 54                	push   %r12 # 保存 r12 寄存器的值
  4010ac:	55                   	push   %rbp # 保存 rbp 寄存器的值
  4010ad:	53                   	push   %rbx # 保存 rbx 寄存器的值
  4010ae:	48 83 ec 58          	sub    $0x58,%rsp # 在栈上分配 0x58 字节的空间
  4010b2:	48 8d 74 24 30       	lea    0x30(%rsp),%rsi # 将 rsi 寄存器指向栈上的第 0x30 个字节

  4010b7:	e8 e3 04 00 00       	callq  40159f <read_six_numbers> # 调用 read_six_numbers 函数,读取 6 个数字并将它们存储在栈上

  # 这一部分的作用为:遍历输入的 6 个数字,确保它们都小于等于 6 且不相等
  4010bc:	4c 8d 6c 24 30       	lea    0x30(%rsp),%r13 # 将 r13 寄存器指向栈上的第 0x30 个字节,即输入的 6 个数字
  4010c1:	41 bc 00 00 00 00    	mov    $0x0,%r12d # 初始化 r12d 寄存器为 0,作为游标
  4010c7:	4c 89 ed             	mov    %r13,%rbp # 将 rbp 寄存器指向 r13 所指向的位置
  4010ca:	41 8b 45 00          	mov    0x0(%r13),%eax # 将输入的第一个数字加载到 eax 寄存器
  4010ce:	83 e8 01             	sub    $0x1,%eax # 将 eax 寄存器中的值减 1
  4010d1:	83 f8 05             	cmp    $0x5,%eax # 比较 eax 寄存器中的值是否小于等于 5
  4010d4:	76 05                	jbe    4010db <phase_6+0x33> # 如果小于等于 5,跳转到 4010db
  4010d6:	e8 8e 04 00 00       	callq  401569 <explode_bomb> # 否则,调用 explode_bomb 函数引爆炸弹

  4010db:	41 83 c4 01          	add    $0x1,%r12d # 将 r12d 寄存器中的值加 1,指向下一个数字
  4010df:	41 83 fc 06          	cmp    $0x6,%r12d # 比较 r12d 寄存器中的值是否等于 6
  4010e3:	75 07                	jne    4010ec <phase_6+0x44> # 如果不等于 6,跳转到 4010ec
  4010e5:	be 00 00 00 00       	mov    $0x0,%esi # 将 esi 寄存器初始化为 0
  4010ea:	eb 42                	jmp    40112e <phase_6+0x86> # 跳转到 40112e 处继续执行

  4010ec:	44 89 e3             	mov    %r12d,%ebx # 将 r12d 寄存器的值复制到 ebx 寄存器
  4010ef:	48 63 c3             	movslq %ebx,%rax # 将 ebx 寄存器的值扩展为 64 位,存储在 rax 寄存器
  4010f2:	8b 44 84 30          	mov    0x30(%rsp,%rax,4),%eax # 将输入的第 ebx 个数字加载到 eax 寄存器
  4010f6:	39 45 00             	cmp    %eax,0x0(%rbp) # 比较 eax 寄存器中的值与 rbp 寄存器所指向的值是否相等
  4010f9:	75 05                	jne    401100 <phase_6+0x58> # 如果不相等,跳转到 401100
  4010fb:	e8 69 04 00 00       	callq  401569 <explode_bomb> # 否则,调用 explode_bomb 函数引爆炸弹
  401100:	83 c3 01             	add    $0x1,%ebx # 将 ebx 寄存器中的值加 1,指向下一个数字
  401103:	83 fb 05             	cmp    $0x5,%ebx # 比较 ebx 寄存器中的值是否小于等于 5
  401106:	7e e7                	jle    4010ef <phase_6+0x47> # 如果小于等于 5,跳转到 4010ef 继续检查

  401108:	49 83 c5 04          	add    $0x4,%r13 # 将 r13 寄存器指向下一组 6 个数字
  40110c:	eb b9                	jmp    4010c7 <phase_6+0x1f> # 跳转到 4010c7 继续处理下一组 6 个数字

  # 对结点进行操作等
  40110e:	48 8b 52 08          	mov    0x8(%rdx),%rdx # 将 rdx 寄存器中的值加载到 rdx 寄存器
  401112:	83 c0 01             	add    $0x1,%eax # 将 eax 寄存器中的值加 1
  401115:	39 c8                	cmp    %ecx,%eax # 比较 eax 寄存器中的值与 ecx 寄存器中的值
  401117:	75 f5                	jne    40110e <phase_6+0x66> # 如果不相等,跳转到 40110e 继续处理
  401119:	eb 05                	jmp    401120 <phase_6+0x78> # 跳转到 401120

  40111b:	ba f0 42 60 00       	mov    $0x6042f0,%edx # 将 0x6042f0 加载到 edx 寄存器
  401120:	48 89 14 74          	mov    %rdx,(%rsp,%rsi,2) # 将 rdx 寄存器中的值存储到栈上的地址中
  401124:	48 83 c6 04          	add    $0x4,%rsi # 将 rsi 寄存器中的值加 4
  401128:	48 83 fe 18          	cmp    $0x18,%rsi # 比较 rsi 寄存器中的值是否等于 0x18
  40112c:	74 15                	je     401143 <phase_6+0x9b> # 如果等于 0x18,跳转到 401143

  40112e:	8b 4c 34 30          	mov    0x30(%rsp,%rsi,1),%ecx # 将栈上的值加载到 ecx 寄存器
  401132:	83 f9 01             	cmp    $0x1,%ecx # 比较 ecx 寄存器中的值是否小于等于 1
  401135:	7e e4                	jle    40111b <phase_6+0x73> # 如果小于等于 1,跳转到 40111b

  401137:	b8 01 00 00 00       	mov    $0x1,%eax # 将 eax 寄存器初始化为 1
  40113c:	ba f0 42 60 00       	mov    $0x6042f0,%edx # 将 0x6042f0 加载到 edx 寄存器
  401141:	eb cb                	jmp    40110e <phase_6+0x66> # 跳转到 40110e 继续处理

  401143:	48 8b 1c 24          	mov    (%rsp),%rbx # 将栈顶的值加载到 rbx 寄存器
  401147:	48 8d 44 24 08       	lea    0x8(%rsp),%rax # 将 rsp+8 的地址加载到 rax 寄存器
  40114c:	48 8d 74 24 30       	lea    0x30(%rsp),%rsi # 将 rsp+0x30 的地址加载到 rsi 寄存器
  401151:	48 89 d9             	mov    %rbx,%rcx # 将 rbx 寄存器的值复制到 rcx 寄存器
  401154:	48 8b 10             	mov    (%rax),%rdx # 将 rax 寄存器所指向的值加载到 rdx 寄存器
  401157:	48 89 51 08          	mov    %rdx,0x8(%rcx) # 将 rdx 寄存器的值存储到 rcx+8 的地址
  40115b:	48 83 c0 08          	add    $0x8,%rax # 将 rax 寄存器中的值加 8
  40115f:	48 39 f0             	cmp    %rsi,%rax # 比较 rax 寄存器与 rsi 寄存器的值
  401162:	74 05                	je     401169 <phase_6+0xc1> # 如果相等,跳转到 401169
  401164:	48 89 d1             	mov    %rdx,%rcx # 将 rdx 寄存器的值复制到 rcx 寄存器
  401167:	eb eb                	jmp    401154 <phase_6+0xac> # 跳转到 401154 继续处理

  401169:	48 c7 42 08 00 00 00 	movq   $0x0,0x8(%rdx) # 将 0 存储到 rdx+8 的地址
  401170:	00 
  401171:	bd 05 00 00 00       	mov    $0x5,%ebp # 将 ebp 寄存器初始化为 5
  401176:	48 8b 43 08          	mov    0x8(%rbx),%rax # 将 rbx+8 的地址中的值加载到 rax 寄存器
  40117a:	8b 00                	mov    (%rax),%eax # 将 rax 寄存器所指向的值加载到 eax 寄存器
  40117c:	39 03                	cmp    %eax,(%rbx) # 比较 rbx 寄存器所指向的值与 eax 寄存器中的值
  40117e:	7e 05                	jle    401185 <phase_6+0xdd> # 如果小于等于,跳转到 401185
  401180:	e8 e4 03 00 00       	callq  401569 <explode_bomb> # 否则,调用 explode_bomb 函数引爆炸弹

  401185:	48 8b 5b 08          	mov    0x8(%rbx),%rbx # 将 rbx+8 的地址中的值加载到 rbx 寄存器
  401189:	83 ed 01             	sub    $0x1,%ebp # 将 ebp 寄存器中的值减 1
  40118c:	75 e8                	jne    401176 <phase_6+0xce> # 如果 ebp 寄存器中的值不为 0,跳转到 401176 继续处理

  40118e:	48 83 c4 58          	add    $0x58,%rsp # 将 rsp 寄存器加 0x58,释放栈空间
  401192:	5b                   	pop    %rbx # 恢复 rbx 寄存器的值
  401193:	5d                   	pop    %rbp # 恢复 rbp 寄存器的值
  401194:	41 5c                	pop    %r12 # 恢复 r12 寄存器的值
  401196:	41 5d                	pop    %r13 # 恢复 r13 寄存器的值
  401198:	c3                   	retq   # 返回
```

### 解答过程

获取结点的数值

```cmd
(gdb) x /24w 0x6042f0
0x6042f0 <node1>:	822	1	6308608	0
0x604300 <node2>:	643	2	6308624	0
0x604310 <node3>:	468	3	6308640	0
0x604320 <node4>:	949	4	6308656	0
0x604330 <node5>:	427	5	6308672	0
0x604340 <node6>:	672	6	0	0
```

通过分析代码可知，应使得链表从小到大排列

故答案为`5 3 2 6 1 4`

---

## 进入secret_phase

我们在`phase_defused`上打个断点看看汇编
```cmd
(gdb) disas phase_defused
Dump of assembler code for function phase_defused:
   0x0000000000401777 <+0>:	sub    $0x68,%rsp
   0x000000000040177b <+4>:	mov    $0x1,%edi
   0x0000000000401780 <+9>:	callq  0x4014ba <send_msg>
   0x0000000000401785 <+14>:	cmpl   $0x6,0x203010(%rip)        # 0x60479c <num_input_strings>
   0x000000000040178c <+21>:	jne    0x4017fb <phase_defused+132>
   
   0x000000000040178e <+23>:	lea    0x10(%rsp),%r8
   0x0000000000401793 <+28>:	lea    0x8(%rsp),%rcx
   0x0000000000401798 <+33>:	lea    0xc(%rsp),%rdx
   
   0x000000000040179d <+38>:	mov    $0x40284b,%esi
   0x00000000004017a2 <+43>:	mov    $0x6048b0,%edi
   0x00000000004017a7 <+48>:	mov    $0x0,%eax
   0x00000000004017ac <+53>:	callq  0x400c30 <__isoc99_sscanf@plt>
   0x00000000004017b1 <+58>:	cmp    $0x3,%eax
   0x00000000004017b4 <+61>:	jne    0x4017e7 <phase_defused+112>
   
   0x00000000004017b6 <+63>:	mov    $0x402854,%esi
   0x00000000004017bb <+68>:	lea    0x10(%rsp),%rdi
   0x00000000004017c0 <+73>:	callq  0x40132e <strings_not_equal>
   
   0x00000000004017c5 <+78>:	test   %eax,%eax
   0x00000000004017c7 <+80>:	jne    0x4017e7 <phase_defused+112>
   0x00000000004017c9 <+82>:	mov    $0x4026a0,%edi
   0x00000000004017ce <+87>:	callq  0x400b40 <puts@plt>
   0x00000000004017d3 <+92>:	mov    $0x4026c8,%edi
   0x00000000004017d8 <+97>:	callq  0x400b40 <puts@plt>
   0x00000000004017dd <+102>:	mov    $0x0,%eax
   
   0x00000000004017e2 <+107>:	callq  0x401248 <secret_phase> #跳转secret_phase
   
   0x00000000004017e7 <+112>:	mov    $0x402700,%edi
   0x00000000004017ec <+117>:	callq  0x400b40 <puts@plt>
   0x00000000004017f1 <+122>:	mov    $0x402730,%edi
   0x00000000004017f6 <+127>:	callq  0x400b40 <puts@plt>
   0x00000000004017fb <+132>:	add    $0x68,%rsp
   0x00000000004017ff <+136>:	retq   
End of assembler dump.
```
找到`secret_phase`的进入条件
```assembly
  0x00000000004017e2 <+107>:	callq  0x401248 <secret_phase>
```
检查整个函数，发现没有任何一条跳转语句直接跳到`4017e2`和前一个地址，所以进入方法是满足在`4017e2`前不跳转
```assembly
   0x0000000000401785 <+14>:	cmpl   $0x6,0x203010(%rip)        # 0x60479c <num_input_strings>
   0x000000000040178c <+21>:	jne    0x4017fb <phase_defused+132>
```
 由此可知在整个程序中必须已经读入六个字符串，即通过了前面的所有phase
```assembly
   0x000000000040179d <+38>:	mov    $0x40284b,%esi
   0x00000000004017a2 <+43>:	mov    $0x6048b0,%edi
   0x00000000004017a7 <+48>:	mov    $0x0,%eax
   0x00000000004017ac <+53>:	callq  0x400c30 <__isoc99_sscanf@plt>
   0x00000000004017b1 <+58>:	cmp    $0x3,%eax
   0x00000000004017b4 <+61>:	jne    0x4017e7 <phase_defused+112>
```
由此可知需要读入三个参数，打印一下`0x40284b`里的格式串

```cmd
(gdb) x /s 0x40284b
0x40284b:	"%d %d %s"
```
这个格式是我们在前面从来没有见过的，考虑前面有两个整型，应该是 phase3 或者 phase4再看看这个字符串应该是什么东西
```assembly
   0x00000000004017b6 <+63>:	mov    $0x402854,%esi
   0x00000000004017bb <+68>:	lea    0x10(%rsp),%rdi
   0x00000000004017c0 <+73>:	callq  0x40132e <strings_not_equal>
```
获取这个字符串：打印`0x402854`的值
```cmd
(gdb) x /s 0x402854
0x402854:	"DrEvil"
```
经过尝试得到phase4答案后面加DrEvil进去了

---

## secret_phase

### 汇编代码

#### secret_phase

```
00000000004011d7 <secret_phase>:
  4011d7:	53                   	push   %rbx
  4011d8:	e8 04 04 00 00       	callq  4015e1 <read_line>
  4011dd:	ba 0a 00 00 00       	mov    $0xa,%edx
  4011e2:	be 00 00 00 00       	mov    $0x0,%esi
  4011e7:	48 89 c7             	mov    %rax,%rdi
  4011ea:	e8 11 fa ff ff       	callq  400c00 <strtol@plt>
  4011ef:	48 89 c3             	mov    %rax,%rbx
  4011f2:	8d 40 ff             	lea    -0x1(%rax),%eax
  4011f5:	3d e8 03 00 00       	cmp    $0x3e8,%eax
  4011fa:	76 05                	jbe    401201 <secret_phase+0x2a>
  4011fc:	e8 68 03 00 00       	callq  401569 <explode_bomb>
  401201:	89 de                	mov    %ebx,%esi
  401203:	bf 10 41 60 00       	mov    $0x604110,%edi
  401208:	e8 8c ff ff ff       	callq  401199 <fun7>
  40120d:	83 f8 04             	cmp    $0x4,%eax
  401210:	74 05                	je     401217 <secret_phase+0x40>
  401212:	e8 52 03 00 00       	callq  401569 <explode_bomb>
  401217:	bf f0 24 40 00       	mov    $0x4024f0,%edi
  40121c:	e8 1f f9 ff ff       	callq  400b40 <puts@plt>
  401221:	e8 e1 04 00 00       	callq  401707 <phase_defused>
  401226:	5b                   	pop    %rbx
  401227:	c3                   	retq   
  401228:	0f 1f 84 00 00 00 00 	nopl   0x0(%rax,%rax,1)
  40122f:	00 
```
#### fun7

```assembly
000000000040120a <fun7>:
  40120a:       48 83 ec 08             sub    $0x8,%rsp
  40120e:       48 85 ff                test   %rdi,%rdi
  401211:       74 2b                   je     40123e <fun7+0x34>
  
  401213:       8b 17                   mov    (%rdi),%edx
  401215:       39 f2                   cmp    %esi,%edx
  401217:       7e 0d                   jle    401226 <fun7+0x1c>
  
  401219:       48 8b 7f 08             mov    0x8(%rdi),%rdi
  40121d:       e8 e8 ff ff ff          callq  40120a <fun7>
  
  401222:       01 c0                   add    %eax,%eax
  401224:       eb 1d                   jmp    401243 <fun7+0x39>
  
  401226:       b8 00 00 00 00          mov    $0x0,%eax
  40122b:       39 f2                   cmp    %esi,%edx
  40122d:       74 14                   je     401243 <fun7+0x39>
  
  40122f:       48 8b 7f 10             mov    0x10(%rdi),%rdi
  401233:       e8 d2 ff ff ff          callq  40120a <fun7>
  
  401238:       8d 44 00 01             lea    0x1(%rax,%rax,1),%eax
  40123c:       eb 05                   jmp    401243 <fun7+0x39>
  
  40123e:       b8 ff ff ff ff          mov    $0xffffffff,%eax
  
  401243:       48 83 c4 08             add    $0x8,%rsp
  401247:       c3                      retq
```

### 解答过程

secret_phase向fun7传入参数

fun7的返回值和4比较，不等就爆炸

分析fun7，看到`401219`处，又出现了链式结构，尝试打印第一个传进来的参数`0x604110`附近有什么
```
(gdb) x /128xw 0x604110
0x604110 <n1>:	0x00000024	0x00000000	0x00604130	0x00000000
0x604120 <n1+16>:	0x00604150	0x00000000	0x00000000	0x00000000
0x604130 <n21>:	0x00000008	0x00000000	0x006041b0	0x00000000
0x604140 <n21+16>:	0x00604170	0x00000000	0x00000000	0x00000000
0x604150 <n22>:	0x00000032	0x00000000	0x00604190	0x00000000
0x604160 <n22+16>:	0x006041d0	0x00000000	0x00000000	0x00000000
0x604170 <n32>:	0x00000016	0x00000000	0x00604290	0x00000000
0x604180 <n32+16>:	0x00604250	0x00000000	0x00000000	0x00000000
0x604190 <n33>:	0x0000002d	0x00000000	0x006041f0	0x00000000
0x6041a0 <n33+16>:	0x006042b0	0x00000000	0x00000000	0x00000000
0x6041b0 <n31>:	0x00000006	0x00000000	0x00604210	0x00000000
0x6041c0 <n31+16>:	0x00604270	0x00000000	0x00000000	0x00000000
0x6041d0 <n34>:	0x0000006b	0x00000000	0x00604230	0x00000000
0x6041e0 <n34+16>:	0x006042d0	0x00000000	0x00000000	0x00000000
0x6041f0 <n45>:	0x00000028	0x00000000	0x00000000	0x00000000
0x604200 <n45+16>:	0x00000000	0x00000000	0x00000000	0x00000000
0x604210 <n41>:	0x00000001	0x00000000	0x00000000	0x00000000
0x604220 <n41+16>:	0x00000000	0x00000000	0x00000000	0x00000000
0x604230 <n47>:	0x00000063	0x00000000	0x00000000	0x00000000
0x604240 <n47+16>:	0x00000000	0x00000000	0x00000000	0x00000000
0x604250 <n44>:	0x00000023	0x00000000	0x00000000	0x00000000
0x604260 <n44+16>:	0x00000000	0x00000000	0x00000000	0x00000000
0x604270 <n42>:	0x00000007	0x00000000	0x00000000	0x00000000
0x604280 <n42+16>:	0x00000000	0x00000000	0x00000000	0x00000000
0x604290 <n43>:	0x00000014	0x00000000	0x00000000	0x00000000
0x6042a0 <n43+16>:	0x00000000	0x00000000	0x00000000	0x00000000
0x6042b0 <n46>:	0x0000002f	0x00000000	0x00000000	0x00000000
0x6042c0 <n46+16>:	0x00000000	0x00000000	0x00000000	0x00000000
0x6042d0 <n48>:	0x000003e9	0x00000000	0x00000000	0x00000000
0x6042e0 <n48+16>:	0x00000000	0x00000000	0x00000000	0x00000000

0x6042f0 <node1>:	0x000001a2	0x00000001	0x00604310	0x00000000
0x604300 <node2>:	0x0000038b	0x00000002	0x00604330	0x00000000
```

显然，前面存的是一个value，后面存的两个地址，一个数据域两个指针域，猜想是一棵二叉树的节点
尝试画出这棵树

```
              36
            /    \
          /        \
        /            \
      8               50
    /    \          /    \
   /      \        /      \
  6       22      45      107
 / \     /  \    /  \    /   \
1   7   20  35  40  47  99   1001
```

翻译为C语言代码

```c
int fun7(node* pNode, int value) {
    if (pNode == NULL) 
        return -1;
    else if (value < pNode->data) 
        return 2 * fun7(pNode->left, value);
    else if (value == pNode->data) 
        return 0;
    else 
        return 2 * fun7(pNode->right, value) + 1;
}
```
调用时传入的节点是根节点，考虑如何凑出4：`2*(2*(2*(return 0)+1))`可以凑出4

即根节点先走两次left再走right到达一个value相等的节点

可得到答案为`7`

---

``` END
至此全部问题解决完毕，未出现爆炸，完结撒花*
```

# 答案总结

1. > I am the mayor. I can do anything I want.

2. > 1 2 4 8 16 32

3. > 2 516

4. > 24 2 DrEvil（DrEvil是进入隐藏关的钥匙）

5. > abgljk

6. > 5 3 2 6 1 4

7. >7
