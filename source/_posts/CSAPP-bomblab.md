---
title: CSAPP - bomblab
date: 2020-02-16 21:06:08
tags:
---

## 安装 GDB

### 问题一

在 Ubuntu 18.04 LTS 中使用 `apt-get install gdb` 安装失败。替换使用国内 Ubuntu 的镜像源后安装成功。

1. 首先更新 `/etc/apt/sources.list`，将该文件内容替换为阿里源：

   ```shell
   deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
   ```

2. 运行 `apt-get update` 和 `apt-get upgrade` 更新。

3. 安装 gdb：`apt-get install gdb`。

### 问题二

执行 gdb 调试时，遇到 Error disabling address space randomization 错误，参考 [warning: Error disabling address space randomization: Operation not permitted](https://stackoverflow.com/questions/35860527/warning-error-disabling-address-space-randomization-operation-not-permitted)，在创建 docker 容器时添加 `--security-opt seccomp=unconfined` 参数：

```shell
docker run -it --cap-add=SYS_PTRACE --security-opt seccomp=unconfined ubuntu bash
```

## 题目

### phase_1

调用 `disassemble phase_1` 反汇编函数 phase_1 ：

```asm
0x400ee0 <+0>:     sub    $0x8,%rsp
0x400ee4 <+4>:     mov    $0x402400,%esi
0x400ee9 <+9>:     callq  0x401338 <strings_not_equal>
0x400eee <+14>:    test   %eax,%eax
0x400ef0 <+16>:    je     0x400ef7 <phase_1+23>
0x400ef2 <+18>:    callq  0x40143a <explode_bomb>
0x400ef7 <+23>:    add    $0x8,%rsp
0x400efb <+27>:    retq
```

显然，该函数调用了函数 `strings_not_equal`，判断两个字符串是否相同。第一个字符串参数 %rdi  为输入 `input` 的地址，第二个参数为比较字符串的地址 0x402400。

在 gdb 中调用 `print (char *) 0x402400` 输出该地址的字符串：

```assembly
Border relations with Canada have never been better.
```

因此 bomb 的第 1 组运行参数为上述字符串。

### phase_2

调用 `disassemble phase_2` 反汇编函数 phase_2 ：

```assembly
0x400efc <+0>:     push   %rbp
0x400efd <+1>:     push   %rbx
0x400efe <+2>:     sub    $0x28,%rsp
0x400f02 <+6>:     mov    %rsp,%rsi
0x400f05 <+9>:     callq  0x40145c <read_six_numbers>
0x400f0a <+14>:    cmpl   $0x1,(%rsp)
0x400f0e <+18>:    je     0x400f30 <phase_2+52>
0x400f10 <+20>:    callq  0x40143a <explode_bomb>
0x400f15 <+25>:    jmp    0x400f30 <phase_2+52>
0x400f17 <+27>:    mov    -0x4(%rbx),%eax
0x400f1a <+30>:    add    %eax,%eax
0x400f1c <+32>:    cmp    %eax,(%rbx)
0x400f1e <+34>:    je     0x400f25 <phase_2+41>
0x400f20 <+36>:    callq  0x40143a <explode_bomb>
0x400f25 <+41>:    add    $0x4,%rbx
0x400f29 <+45>:    cmp    %rbp,%rbx
0x400f2c <+48>:    jne    0x400f17 <phase_2+27>
0x400f2e <+50>:    jmp    0x400f3c <phase_2+64>
0x400f30 <+52>:    lea    0x4(%rsp),%rbx
0x400f35 <+57>:    lea    0x18(%rsp),%rbp
0x400f3a <+62>:    jmp    0x400f17 <phase_2+27>
0x400f3c <+64>:    add    $0x28,%rsp
0x400f40 <+68>:    pop    %rbx
0x400f41 <+69>:    pop    %rbp
0x400f42 <+70>:    retq
```

最先调用了函数 `read_six_numbers`，其起始地址为 0x40145c，调用 `disassemble 0x40145c` 反汇编其代码：

```assembly
0x40145c <+0>:     sub    $0x18,%rsp
0x401460 <+4>:     mov    %rsi,%rdx
0x401463 <+7>:     lea    0x4(%rsi),%rcx
0x401467 <+11>:    lea    0x14(%rsi),%rax
0x40146b <+15>:    mov    %rax,0x8(%rsp)
0x401470 <+20>:    lea    0x10(%rsi),%rax
0x401474 <+24>:    mov    %rax,(%rsp)
0x401478 <+28>:    lea    0xc(%rsi),%r9
0x40147c <+32>:    lea    0x8(%rsi),%r8
0x401480 <+36>:    mov    $0x4025c3,%esi
0x401485 <+41>:    mov    $0x0,%eax
0x40148a <+46>:    callq  0x400bf0 <__isoc99_sscanf@plt>
0x40148f <+51>:    cmp    $0x5,%eax
0x401492 <+54>:    jg     0x401499 <read_six_numbers+61>
0x401494 <+56>:    callq  0x40143a <explode_bomb>
0x401499 <+61>:    add    $0x18,%rsp
0x40149d <+65>:    retq
```

先注意该函数调用了 `sscanf` 函数，其从一个字符串而不是标准输入流读取输入。`sscanf` 的第二个参数为输入格式，尝试打印其字符串（0x4025c3）：

```assembly
$1 = 0x4025c3 "%d %d %d %d %d %d"
```

与函数名相应，该函数从输入 `input` 中读取 6 个整数。

由于`sscanf` 参数超过 6 个，前 6 个参数**按序**分别存储于 %edi, %esi, %rdx, %rcx, %r8, %r9，剩余两个参数存储于栈中，其中第 7 个参数位于栈顶。阅读代码可知 6 个整数按序存储于 %rsi, %rsi + 4, ..., %rsi + 20。

0x400f0a 行的代码表示读入的第 1 个数必须等于 1。

从 0x400f17 到 0x400f2c 的代码表示，\*(rbp) = \*(rbp - 0x4) \* 2，即读入的 6 个整数表示首项为 1 公比为 2 的等比数列： 

```assembly
1 2 4 8 16 32
```

该序列为 bomb 的第 2 组运行参数。

### phase_3

调用 `disassemble phase_3` 反汇编函数 phase_3 ：

```assembly
0x400f43 <+0>:     sub    $0x18,%rsp
0x400f47 <+4>:     lea    0xc(%rsp),%rcx
0x400f4c <+9>:     lea    0x8(%rsp),%rdx
0x400f51 <+14>:    mov    $0x4025cf,%esi
0x400f56 <+19>:    mov    $0x0,%eax
0x400f5b <+24>:    callq  0x400bf0 <__isoc99_sscanf@plt>
0x400f60 <+29>:    cmp    $0x1,%eax
0x400f63 <+32>:    jg     0x400f6a <phase_3+39>
0x400f65 <+34>:    callq  0x40143a <explode_bomb>
0x400f6a <+39>:    cmpl   $0x7,0x8(%rsp)
0x400f6f <+44>:    ja     0x400fad <phase_3+106>
0x400f71 <+46>:    mov    0x8(%rsp),%eax
0x400f75 <+50>:    jmpq   *0x402470(,%rax,8)
0x400f7c <+57>:    mov    $0xcf,%eax
0x400f81 <+62>:    jmp    0x400fbe <phase_3+123>
0x400f83 <+64>:    mov    $0x2c3,%eax
0x400f88 <+69>:    jmp    0x400fbe <phase_3+123>
0x400f8a <+71>:    mov    $0x100,%eax
0x400f8f <+76>:    jmp    0x400fbe <phase_3+123>
0x400f91 <+78>:    mov    $0x185,%eax
0x400f96 <+83>:    jmp    0x400fbe <phase_3+123>
0x400f98 <+85>:    mov    $0xce,%eax
0x400f9d <+90>:    jmp    0x400fbe <phase_3+123>
0x400f9f <+92>:    mov    $0x2aa,%eax
0x400fa4 <+97>:    jmp    0x400fbe <phase_3+123>
0x400fa6 <+99>:    mov    $0x147,%eax
0x400fab <+104>:   jmp    0x400fbe <phase_3+123>
0x400fad <+106>:   callq  0x40143a <explode_bomb>
0x400fb2 <+111>:   mov    $0x0,%eax
0x400fb7 <+116>:   jmp    0x400fbe <phase_3+123>
0x400fb9 <+118>:   mov    $0x137,%eax
0x400fbe <+123>:   cmp    0xc(%rsp),%eax
0x400fc2 <+127>:   je     0x400fc9 <phase_3+134>
0x400fc4 <+129>:   callq  0x40143a <explode_bomb>
0x400fc9 <+134>:   add    $0x18,%rsp
0x400fcd <+138>:   retq
```

首先看函数 `sscanf` 的调用：0x4025cf 地址字符串为 `$2 = 0x4025cf "%d %d"`，从 `input` 读取两个整数，分别存储于地址 0x8(%rsp) 和 0xc(%rsp)。

`cmpl $0x7,0x8(%rsp)`  和 `ja 0x400fad <phase_3+106>` 表明，第一个整数的值不能超过 7。

`jmpq *0x402470(,%rax,8)` 表示跳转到 *(0x402470 + 8 * %rax) 处，而 %rax 的值此时等于读取的第一个整数。假设输入的第一个整数为 0，下一个指令的地址为 0x402470，使用 `print /x *0x402470` 打印出地址的值为 0x400f7c，即指令 `jmpq *0x402470(,%rax,8)` 等同于 `jmpq 0x400f7c `。地址为 0x400f7c 的指令为 `mov $0xcf,%eax`，随后跳转到 `cmp 0xc(%rsp),%eax`，即与输入的第二个整数比较，相同时结束该函数调用。说明当输入为 0 207 时（0xcf = 207），`phase_3` 不会爆炸。

对于第 1 个整数分别为 1~7 时，其推理过程也是类似的，其结果如下表：

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;border:none;border-color:#aaa;}
.tg td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-color:#aaa;color:#333;background-color:#fff;}
.tg th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-color:#aaa;color:#fff;background-color:#f38630;}
.tg .tg-vswx{background-color:#fd6864;border-color:inherit;text-align:center;vertical-align:top}
.tg .tg-c3ow{border-color:inherit;text-align:center;vertical-align:top}
</style>
<table class="tg">
  <tr>
    <th class="tg-vswx">数1</th>
    <th class="tg-vswx">下条指令地址</th>
    <th class="tg-vswx">跳转地址</th>
    <th class="tg-vswx">数2（16进制）</th>
    <th class="tg-vswx">数2（10进制）</th>
  </tr>
  <tr>
    <td class="tg-c3ow">0</td>
    <td class="tg-c3ow">0x402470</td>
    <td class="tg-c3ow">0x400f7c</td>
    <td class="tg-c3ow">0xcf</td>
    <td class="tg-c3ow">207</td>
  </tr>
  <tr>
    <td class="tg-c3ow">1</td>
    <td class="tg-c3ow">0x402478</td>
    <td class="tg-c3ow">0x400fb9</td>
    <td class="tg-c3ow">0x137</td>
    <td class="tg-c3ow">311</td>
  </tr>
  <tr>
    <td class="tg-c3ow">2</td>
    <td class="tg-c3ow">0x402480</td>
    <td class="tg-c3ow">0x400f83</td>
    <td class="tg-c3ow">0x2c3</td>
    <td class="tg-c3ow">707</td>
  </tr>
  <tr>
    <td class="tg-c3ow">3</td>
    <td class="tg-c3ow">0x402488</td>
    <td class="tg-c3ow">0x400f8a</td>
    <td class="tg-c3ow">0x100</td>
    <td class="tg-c3ow">256</td>
  </tr>
  <tr>
    <td class="tg-c3ow">4</td>
    <td class="tg-c3ow">0x402490</td>
    <td class="tg-c3ow">0x400f91</td>
    <td class="tg-c3ow">0x185</td>
    <td class="tg-c3ow">389</td>
  </tr>
  <tr>
    <td class="tg-c3ow">5</td>
    <td class="tg-c3ow">0x402498</td>
    <td class="tg-c3ow">0x400f98</td>
    <td class="tg-c3ow">0xce</td>
    <td class="tg-c3ow">206</td>
  </tr>
  <tr>
    <td class="tg-c3ow">6</td>
    <td class="tg-c3ow">0x4024a0</td>
    <td class="tg-c3ow">0x400f9f</td>
    <td class="tg-c3ow">0x2aa</td>
    <td class="tg-c3ow">682</td>
  </tr>
</table>

每对<数1，数2>均可作为 bomb 的第 3 组参数。

### phase_4

调用 `disassemble phase_4` 反汇编函数 phase_4：

```assembly
0x40100c <+0>:     sub    $0x18,%rsp
0x401010 <+4>:     lea    0xc(%rsp),%rcx
0x401015 <+9>:     lea    0x8(%rsp),%rdx
0x40101a <+14>:    mov    $0x4025cf,%esi
0x40101f <+19>:    mov    $0x0,%eax
0x401024 <+24>:    callq  0x400bf0 <__isoc99_sscanf@plt>
0x401029 <+29>:    cmp    $0x2,%eax
0x40102c <+32>:    jne    0x401035 <phase_4+41>
0x40102e <+34>:    cmpl   $0xe,0x8(%rsp)
0x401033 <+39>:    jbe    0x40103a <phase_4+46>
0x401035 <+41>:    callq  0x40143a <explode_bomb>
0x40103a <+46>:    mov    $0xe,%edx
0x40103f <+51>:    mov    $0x0,%esi
0x401044 <+56>:    mov    0x8(%rsp),%edi
0x401048 <+60>:    callq  0x400fce <func4>
0x40104d <+65>:    test   %eax,%eax
0x40104f <+67>:    jne    0x401058 <phase_4+76>
0x401051 <+69>:    cmpl   $0x0,0xc(%rsp)
0x401056 <+74>:    je     0x40105d <phase_4+81>
0x401058 <+76>:    callq  0x40143a <explode_bomb>
0x40105d <+81>:    add    $0x18,%rsp
0x401061 <+85>:    retq
```

1. 首先看读入部分，相关代码如下：

   ```assembly
   0x40100c <+0>:     sub    $0x18,%rsp
   0x401010 <+4>:     lea    0xc(%rsp),%rcx
   0x401015 <+9>:     lea    0x8(%rsp),%rdx
   0x40101a <+14>:    mov    $0x4025cf,%esi
   0x40101f <+19>:    mov    $0x0,%eax
   0x401024 <+24>:    callq  0x400bf0 <__isoc99_sscanf@plt>
   0x401029 <+29>:    cmp    $0x2,%eax
   0x40102c <+32>:    jne    0x401035 <phase_4+41>
   ```

   函数 sscanf 的参数一为 %rdi，存储输入 input 的地址；参数二为 %esi，使用 `(gdb) print （char *) 0x4025cf` 查看该地址的字符串，其值为 "%d %d" 表示读入两个整数；参数三为 %rdx=%rsp+0x8, 参数四为 %rcx=%rsp + %c，说明读入的两个整数分别存储于地址 %rsp+0x8 和 %rsp+0xc。

   %eax 表示读入的整数数量，如果不为 2 则跳转到调用 explode_bomb 函数的指令地址。

2. 接下来，程序准备调用函数 func4：

   ```assembly
   0x40102e <+34>:    cmpl   $0xe,0x8(%rsp)
   0x401033 <+39>:    jbe    0x40103a <phase_4+46>
   0x401035 <+41>:    callq  0x40143a <explode_bomb>
   0x40103a <+46>:    mov    $0xe,%edx
   0x40103f <+51>:    mov    $0x0,%esi
   0x401044 <+56>:    mov    0x8(%rsp),%edi
   0x401048 <+60>:    callq  0x400fce <func4>
   ```

   0x40102e~0x401035（1~3 行）：要求读入的第一个整数必须 ≤14。

   0x40103a~0x401044（4~6 行）：构造 func4 参数：第一个整数，0，14，即调用函数 func4(数一，0，14)。

3. 处理函数 func4 的返回值：

   ```assembly
   0x40104d <+65>:    test   %eax,%eax
   0x40104f <+67>:    jne    0x401058 <phase_4+76>
   ```

   如果 func4() 返回值不等于 0，则跳转并调用 explode_bomb。

4. 处理读入的第二个整数：

   ```assembly
   0x401051 <+69>:    cmpl   $0x0,0xc(%rsp)
   0x401056 <+74>:    je     0x40105d <phase_4+81>
   0x401058 <+76>:    callq  0x40143a <explode_bomb>
   0x40105d <+81>:    add    $0x18,%rsp
   0x401061 <+85>:    retq
   ```

   如果**数二等于 0**，则跳转指令并结束；否则调用 explode_bomb。

5. 调用 `disassemble func4 ` 反汇编函数 func4：

   ```assembly
   0x400fce <+0>:     sub    $0x8,%rsp
   0x400fd2 <+4>:     mov    %edx,%eax
   0x400fd4 <+6>:     sub    %esi,%eax
   0x400fd6 <+8>:     mov    %eax,%ecx
   0x400fd8 <+10>:    shr    $0x1f,%ecx
   0x400fdb <+13>:    add    %ecx,%eax
   0x400fdd <+15>:    sar    %eax
   0x400fdf <+17>:    lea    (%rax,%rsi,1),%ecx
   0x400fe2 <+20>:    cmp    %edi,%ecx
   0x400fe4 <+22>:    jle    0x400ff2 <func4+36>
   0x400fe6 <+24>:    lea    -0x1(%rcx),%edx
   0x400fe9 <+27>:    callq  0x400fce <func4>
   0x400fee <+32>:    add    %eax,%eax
   0x400ff0 <+34>:    jmp    0x401007 <func4+57>
   0x400ff2 <+36>:    mov    $0x0,%eax
   0x400ff7 <+41>:    cmp    %edi,%ecx
   0x400ff9 <+43>:    jge    0x401007 <func4+57>
   0x400ffb <+45>:    lea    0x1(%rcx),%esi
   0x400ffe <+48>:    callq  0x400fce <func4>
   0x401003 <+53>:    lea    0x1(%rax,%rax,1),%eax
   0x401007 <+57>:    add    $0x8,%rsp
   0x40100b <+61>:    retq
   ```

   在第 2 步中已知 func4 的三个参数为整数，因此这里将 func4 定义为 `int func4(int a, int b, int c)`，其中 %edi = a, %esi = b, %edx = c。

   0x400fd2~0x400fdf（2~8 行）：%eax = (c - b) / 2, %ecx = b + (c - b) / 2。记 m = b + (c - b) / 2，为区间 [b, c] 的中点。

   0x400fe2~0x400fe4（9-10 行）：如果 %ecx <= %edi（即 m <= a），则跳转到 0x400ff2。

   a. 不跳转时，执行 0x400fe6~0x400ff0（11~14 行）：调用并返回结果值 2 * func4(a, b, m - 1)。

   b. 发生跳转，执行 0x400ff2~0x400ff9（15~17 行）：如果 %ecx >= %edi（即 m >= a），返回结果值 0，否咋调用并返回 2 * func(a, m + 1, c) + 1。

   依据第 3 步的推论，func4(数 1, 0, 14) 的结果必须为 0，因此**数一等于 7**。

因此 7 0 为 bomb 的第 4 组运行参数。

### phase_5

调用 `disassemble phase_5` 反汇编函数 phase_5

```assembly
0x401062 <+0>:     push   %rbx
0x401063 <+1>:     sub    $0x20,%rsp
0x401067 <+5>:     mov    %rdi,%rbx
0x40106a <+8>:     mov    %fs:0x28,%rax
0x401073 <+17>:    mov    %rax,0x18(%rsp)
0x401078 <+22>:    xor    %eax,%eax
0x40107a <+24>:    callq  0x40131b <string_length>
0x40107f <+29>:    cmp    $0x6,%eax
0x401082 <+32>:    je     0x4010d2 <phase_5+112>
0x401084 <+34>:    callq  0x40143a <explode_bomb>
0x401089 <+39>:    jmp    0x4010d2 <phase_5+112>
0x40108b <+41>:    movzbl (%rbx,%rax,1),%ecx
0x40108f <+45>:    mov    %cl,(%rsp)
0x401092 <+48>:    mov    (%rsp),%rdx
0x401096 <+52>:    and    $0xf,%edx
0x401099 <+55>:    movzbl 0x4024b0(%rdx),%edx
0x4010a0 <+62>:    mov    %dl,0x10(%rsp,%rax,1)
0x4010a4 <+66>:    add    $0x1,%rax
0x4010a8 <+70>:    cmp    $0x6,%rax
0x4010ac <+74>:    jne    0x40108b <phase_5+41>
0x4010ae <+76>:    movb   $0x0,0x16(%rsp)
0x4010b3 <+81>:    mov    $0x40245e,%esi
0x4010b8 <+86>:    lea    0x10(%rsp),%rdi
0x4010bd <+91>:    callq  0x401338 <strings_not_equal>
0x4010c2 <+96>:    test   %eax,%eax
0x4010c4 <+98>:    je     0x4010d9 <phase_5+119>
0x4010c6 <+100>:   callq  0x40143a <explode_bomb>
0x4010cb <+105>:   nopl   0x0(%rax,%rax,1)
0x4010d0 <+110>:   jmp    0x4010d9 <phase_5+119>
0x4010d2 <+112>:   mov    $0x0,%eax
0x4010d7 <+117>:   jmp    0x40108b <phase_5+41>
0x4010d9 <+119>:   mov    0x18(%rsp),%rax
0x4010de <+124>:   xor    %fs:0x28,%rax
0x4010e7 <+133>:   je     0x4010ee <phase_5+140>
0x4010e9 <+135>:   callq  0x400b30 <__stack_chk_fail@plt>
0x4010ee <+140>:   add    $0x20,%rsp
0x4010f2 <+144>:   pop    %rbx
0x4010f3 <+145>:   retq
```

A. 先阅读读入部分：

```assembly
0x401062 <+0>:     push   %rbx
0x401063 <+1>:     sub    $0x20,%rsp
0x401067 <+5>:     mov    %rdi,%rbx
0x40106a <+8>:     mov    %fs:0x28,%rax
0x401073 <+17>:    mov    %rax,0x18(%rsp)
0x401078 <+22>:    xor    %eax,%eax
0x40107a <+24>:    callq  0x40131b <string_length>
0x40107f <+29>:    cmp    $0x6,%eax
0x401082 <+32>:    je     0x4010d2 <phase_5+112>
0x401084 <+34>:    callq  0x40143a <explode_bomb>
0x401089 <+39>:    jmp    0x4010d2 <phase_5+112>
```

寄存器 %rbx = %rdi = &input，记录输入字符串的地址。

调用函数 string_length 计算输入 %rdi = &input 的字符串长度，要求其长度为 6。

B. 上述代码跳转到 0x4010d2：

```assembly
0x4010d2 <+112>:   mov    $0x0,%eax
0x4010d7 <+117>:   jmp    0x40108b <phase_5+41>
```

该段代码将 %eax 赋值为 0，并跳转到 0x40108b。

C. 0x40108b~0x4010ac 是一个循环，循环变量为 %rax=0~5，%rbx = &input。

```assembly
0x40108b <+41>:    movzbl (%rbx,%rax,1),%ecx
0x40108f <+45>:    mov    %cl,(%rsp)
0x401092 <+48>:    mov    (%rsp),%rdx
0x401096 <+52>:    and    $0xf,%edx
```

上述代码取出 input 的字符，并将低 4 位赋值给 %edx，即 %edx = input[%rax] & 0xf。

```assembly
0x401099 <+55>:    movzbl 0x4024b0(%rdx),%edx
0x4010a0 <+62>:    mov    %dl,0x10(%rsp,%rax,1)
```

上述代码从地址 0x4024b0 + %rdx 取值，并存储于栈 %rsp + %rax + 16 处。

使用 `print (char *) 0x4024b0` 查看该地址的值为 "maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?" 。

该循环利用上述字符串构造了新的字符串，并存储于起始地址 %rsp + 16。

D. 第 3 步构造的字符串要与地址 0x40245e 的字符串相同。

```assembly
0x4010ae <+76>:    movb   $0x0,0x16(%rsp)
0x4010b3 <+81>:    mov    $0x40245e,%esi
0x4010b8 <+86>:    lea    0x10(%rsp),%rdi
0x4010bd <+91>:    callq  0x401338 <strings_not_equal>
0x4010c2 <+96>:    test   %eax,%eax
0x4010c4 <+98>:    je     0x4010d9 <phase_5+119>
0x4010c6 <+100>:   callq  0x40143a <explode_bomb>
```

使用 `print (char *) 0x40245e` 查看字符串，其值为 “flyers”。

E. 构造字符串的第 i 个字符为 *(0x4024b0 + (input[i] & 0xf))，依据 "flyers" 可反向推导 input 的值为 0x9fe567。然而 input 是作为字符串读入的，因此值 0x9fe567 要构造成可打印字符的序列。本文将每个值或（or）上 0x60，构造出序列 "ionefg"。

“ionefg” 即为 bomb 的第 5 组运行参数。

### phase_6

A. 数据读入使用 read_six_numbers：

```assembly
0x401100 <+12>:    mov    %rsp,%r13
0x401103 <+15>:    mov    %rsp,%rsi
0x401106 <+18>:    callq  0x40145c <read_six_numbers>
0x40110b <+23>:    mov    %rsp,%r14
```

在 phase_2 已知 read_six_numbes 读取六个整数，依次存储于 %rsi, %rsi + 4, ..., %rsi + 20。

定义 `int a[6]`，其中 a = %rsi，则六个整数可表示为 a[0], a[1], ..., a[5]。

B. 二重循环检测六个整数的大小关系，要求**两两不同**且每个数不能大于 6。

```assembly
0x401114 <+32>:    mov    %r13,%rbp
...
0x40114d <+89>:    add    $0x4,%r13
0x401151 <+93>:    jmp    0x401114 <phase_6+32>
```

寄存器 %r13 为外层循环指针，依次指向 &a[0], &a[1], ..., &a[5]。

```assembly
0x40110e <+26>:    mov    $0x0,%r12d
...
0x401128 <+52>:    add    $0x1,%r12d
0x40112c <+56>:    cmp    $0x6,%r12d
0x401130 <+60>:    je     0x401153 <phase_6+95>
0x401132 <+62>:    mov    %r12d,%ebx
```

寄存器 %r12 为外层循环下标，依次赋值为 0, 1, ..., 5。

```assembly
0x401132 <+62>:    mov    %r12d,%ebx
0x401135 <+65>:    movslq %ebx,%rax
...
0x401145 <+81>:    add    $0x1,%ebx
0x401148 <+84>:    cmp    $0x5,%ebx
0x40114b <+87>:    jle    0x401135 <phase_6+65>
```

寄存器 %ebx 为内层循环下标。

```assembly
0x401117 <+35>:    mov    0x0(%r13),%eax			# %eax = a[i]
0x40111b <+39>:    sub    $0x1,%eax
0x40111e <+42>:    cmp    $0x5,%eax
0x401121 <+45>:    jbe    0x401128 <phase_6+52>
0x401123 <+47>:    callq  0x40143a <explode_bomb>	# bomb if a[i] > 6
```

上述代码检查 a[i] - 1 <= 5 是否满足，不满足则调用 explode_bomb。

```assembly
0x401135 <+65>:    movslq %ebx,%rax
0x401138 <+68>:    mov    (%rsp,%rax,4),%eax		# %eax = a[j]
0x40113b <+71>:    cmp    %eax,0x0(%rbp)			
0x40113e <+74>:    jne    0x401145 <phase_6+81>
0x401140 <+76>:    callq  0x40143a <explode_bomb>	# bomb if a[i] == a[j]
```

%ebx 为内层循环下标，因此 %eax = a[j]。

%rsp = %r13 为外层循环指针，因此 cmp 指令比较 a[i], a[j]。

上述代码检测 a[i] != a[j]，不满足则调用 explode_bomb。

C. 更新 a[i] = 7 - a[i]。

```assembly
0x401153 <+95>:    lea    0x18(%rsp),%rsi			# 循环上界 %rsp + 24
0x401158 <+100>:   mov    %r14,%rax					# 循环指针 %rax
0x40115b <+103>:   mov    $0x7,%ecx
0x401160 <+108>:   mov    %ecx,%edx
0x401162 <+110>:   sub    (%rax),%edx
0x401164 <+112>:   mov    %edx,(%rax)				# *%rax = 7 - *%rax
0x401166 <+114>:   add    $0x4,%rax
0x40116a <+118>:   cmp    %rsi,%rax
0x40116d <+121>:   jne    0x401160 <phase_6+108>
```

%rax 是指向数组 a 的指针，因此上述代码完成 a[i] = 7 - a[i]。

D. 构造 value 指针数组，第 i 个 value 指针指向 0x6032d0 + 16(a[i] - 1)。

```assembly
0x40116f <+123>:   mov    $0x0,%esi			# %esi 为偏移量 4i
...
0x40118d <+153>:   add    $0x4,%rsi
0x401191 <+157>:   cmp    $0x18,%rsi
```

寄存器 %esi 为偏移量。

```assembly
0x401176 <+130>:   mov    0x8(%rdx),%rdx
0x40117a <+134>:   add    $0x1,%eax
0x40117d <+137>:   cmp    %ecx,%eax
0x40117f <+139>:   jne    0x401176 <phase_6+130>
0x401181 <+141>:   jmp    0x401188 <phase_6+148>
0x401183 <+143>:   mov    $0x6032d0,%edx
```

上述代码根据 %ecx = a[i] 值，使用 %rdx = *(%rdx + 8) 构造 value 指针，%rdx 初始地址为 0x6032d0。

利用 `print *(int *) 0x6032d0` 查询一系列地址的值，可得下表：

<table style="border-collapse:collapse;border-spacing:0;border-color:#ccc" class="tg"><tr><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#ffffff;background-color:#fd6864;text-align:center;vertical-align:top">addr</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#ffffff;background-color:#fd6864;text-align:center;vertical-align:top">value</th></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#f9f9f9;text-align:center;vertical-align:top">0x6032d0</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#f9f9f9;text-align:center;vertical-align:top">332</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:center;vertical-align:top">0x6032d8</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:center;vertical-align:top">0x6032e0</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#f9f9f9;text-align:center;vertical-align:top">‬0x6032e0</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#f9f9f9;text-align:center;vertical-align:top">‬168</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:center;vertical-align:top">‬0x6032e8</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:center;vertical-align:top">0x6032f0‬</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#f9f9f9;text-align:center;vertical-align:top">0x6032f0‬</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#f9f9f9;text-align:center;vertical-align:top">924</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:center;vertical-align:top">0x6032f8</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:center;vertical-align:top">0x603300</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#f9f9f9;text-align:center;vertical-align:top">0x603300</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#f9f9f9;text-align:center;vertical-align:top">691</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:center;vertical-align:top">0x603308</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:center;vertical-align:top">0x603310</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#f9f9f9;text-align:center;vertical-align:top">0x603310</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#f9f9f9;text-align:center;vertical-align:top">477</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:center;vertical-align:top">0x603318</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:center;vertical-align:top">0x603320</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#f9f9f9;text-align:center;vertical-align:top">0x603320</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#f9f9f9;text-align:center;vertical-align:top">443</td></tr></table>
根据代码以及上述表格可得，%rdx = 0x6032d0 + 16 * (a[i] - 1)。

   ```assembly
0x401188 <+148>:   mov    %rdx,0x20(%rsp,%rsi,2)
   ```

%rsi 为第 i 个元素的偏移量，值为 4i。

value 指针数组的起始地址为 %rsp + 32，第 i 个整数的 value 指针地址为 %rsp + 32 + 8i，存储值 %rdx = 0x6032d0 + 16 * (a[i] - 1)。

E. 构造六个元素的链表。
```assembly
0x4011b0 <+188>:   lea    0x28(%rsp),%rax
...
0x4011c4 <+208>:   add    $0x8,%rax
0x4011c8 <+212>:   cmp    %rsi,%rax
```

寄存器 %rax 是指向 value 指针数组的指针，从第 2 个元素开始。

```assembly
0x4011ab <+183>:   mov    0x20(%rsp),%rbx
0x4011ba <+198>:   mov    %rbx,%rc
0x4011bd <+201>:   mov    (%rax),%rdx
0x4011c0 <+204>:   mov    %rdx,0x8(%rcx)
...
0x4011cd <+217>:   mov    %rdx,%rcx
0x4011d0 <+220>:   jmp    0x4011bd <phase_6+201>
```

第 i 个元素的 next 指针地址为 *(%rcx + 8)，而 %rcx = value[i]，因此 next 的地址等于 *(value[i] + 8) = 0x6032d0 + 16 * (a[i] - 1) + 8，该地址存储 *(%rax) 的值 = value[i + 1] =  0x6032d0 + 16 * (a[i + 1] - 1)。

即该链表顺序链接，第 i 个元素的下一个元素为第 i + 1 个元素。

F. 遍历构造链表并比较相邻元素的大小。

   ```assembly
0x4011da <+230>:   mov    $0x5,%ebp
...
0x4011f2 <+254>:   sub    $0x1,%ebp
0x4011f5 <+257>:   jne    0x4011df <phase_6+235>
   ```
%ebp 存储循环下标。

   ```assembly
0x4011df <+235>:   mov    0x8(%rbx),%rax
   ```

%rbx = value[i]，因此 %rax = *(value[i] + 8) = next + i。

   ```assembly
0x4011e3 <+239>:   mov    (%rax),%eax
   ```

从 *(&amp;next[i]) = value[i + 1] 的地址取值并存于 %eax，即 %eax = value[i + 1]。

   ```assembly
0x4011e5 <+241>:   cmp    %eax,(%rbx)
0x4011e7 <+243>:   jge    0x4011ee <phase_6+250>
0x4011e9 <+245>:   callq  0x40143a <explode_bomb>
   ```

cmp 指令比较 value[i + 1] 和 value[i]。

当 value[i] < value[i + 1] 不发生跳转，从而顺序执行 explode_bomb。所以，链表的元素需要满足单调递减的性质。

查询第 4 步中的表可得，链表序列为 924 691 477 443 332 168，对应的 a[i] 序列为 3 4 5 6 1 2。注意第 3 步中用 7 - a[i] 更新了 a[i] 数组，因此原始的 a[i] 序列为 4 3 2 1 6 5。

"4 3 2 1 6 5" 为 bomb 的第 6 组运行参数。

### 答案

Border relations with Canada have never been better.
1 2 4 8 16 32
7 327
7 0
ionefg
4 3 2 1 6 5

  

