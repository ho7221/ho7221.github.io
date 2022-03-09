---
title: "GCC(GNU Compiler Collection)"
category: OS
excerpt: "From Preprocessing to Loading"
toc: true
toc_sticky: true
toc_label: "Table of Contents"
toc_icon: "bars"
---
![compile process](https://user-images.githubusercontent.com/45323902/157246752-28c63635-31b1-4bc4-975b-d5305594e8ad.png){: .align-center}
위 그림에서 알 수 있듯이 source code에서 executable이 되기까지 4단계로 구성된다. 하나씩 알아보자.  
## 예제코드(main.c)
~~~
#include <stdio.h>
#define MACRO 1;

int main(){
	int a=3;
	int b;
	b=MACRO+a;
	return 0;
}
~~~

## Pre-Process(main.i)
source code인 c파일은 사용자가 사용하기 편하게 만들어져있다. 따라서 컴퓨터를 위해 약간의 전처리를 해주는데 헤더파일(#include)을 포함해주고 매크로 변수(#define)를 바꾸어준다. 이 과정을 gcc의 cpp가 담당한다. 전처리만 하는 명령어는 아래와 같다.  
> gcc -E main.c -o main.i

-E 옵션을 이용해 전처리만 하고 -o는 출력파일의 이름을 지정하는 것이다. {: .notice--info}

~~~
# 1 "main.c"
# 1 "<built-in>"
# 1 "<command-line>"
# 31 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 32 "<command-line>" 2
# 1 "main.c"
# 1 "/usr/include/stdio.h" 1 3 4
# 27 "/usr/include/stdio.h" 3 4
...
typedef unsigned char __u_char;
typedef unsigned short int __u_short;
typedef unsigned int __u_int;
typedef unsigned long int __u_long;
...
# 873 "/usr/include/stdio.h" 3 4

# 2 "main.c" 2

# 4 "main.c"
int main(){
 int a=3;
 int b;
 b=1;+a;
 return 0;
}
~~~
위의 코드를 보면 stdio.h 헤더파일에 있는 변수와 함수의 프로토타입을 추가하고 MACRO를 직접 값을 치환한 것을 확인할 수 있다.
## Compile(main.s)
컴파일은 전처리된 코드를 어셈블리어로 변환하는 것이다. cc1에 의해 진행되고 명령어는 아래와 같다.
> gcc -S main.i -o main.s

~~~
	.file	"main.c"
	.text
	.globl	main
	.type	main, @function
main:
.LFB0:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movl	$3, -8(%rbp)
	movl	$1, -4(%rbp)
	movl	$0, %eax
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	main, .-main
	.ident	"GCC: (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0"
	.section	.note.GNU-stack,"",@progbits
	.section	.note.gnu.property,"a"
	.align 8
	.long	 1f - 0f
	.long	 4f - 1f
	.long	 5
0:
	.string	 "GNU"
1:
	.align 8
	.long	 0xc0000002
	.long	 3f - 2f
2:
	.long	 0x3
3:
	.align 8
4:
~~~
## Assemble(main.o)
이 과정에서는 어셈블리어를 기계어인 오브젝트 코드로 변환해준다. 이 과정을 통해 기계어로 변환되었기 때문에 아키텍처만 동일하다면 어느 환경에서든지 돌아가게 된다. 이 과정은 as가 담당하며 명령어는 아래와 같다.
> as main.s -o main.o

이때부터는 일반 텍스트 파일이 아닌 바이너리(바이트코드)이므로 구조만 확인해보자.
> readelf -S main.o

~~~
There are 12 section headers, starting at offset 0x260:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000000000  00000040
       000000000000001d  0000000000000000  AX       0     0     1
  [ 2] .data             PROGBITS         0000000000000000  0000005d
       0000000000000000  0000000000000000  WA       0     0     1
  [ 3] .bss              NOBITS           0000000000000000  0000005d
       0000000000000000  0000000000000000  WA       0     0     1
  [ 4] .comment          PROGBITS         0000000000000000  0000005d
       000000000000002b  0000000000000001  MS       0     0     1
  [ 5] .note.GNU-stack   PROGBITS         0000000000000000  00000088
       0000000000000000  0000000000000000           0     0     1
  [ 6] .note.gnu.propert NOTE             0000000000000000  00000088
       0000000000000020  0000000000000000   A       0     0     8
  [ 7] .eh_frame         PROGBITS         0000000000000000  000000a8
       0000000000000038  0000000000000000   A       0     0     8
  [ 8] .rela.eh_frame    RELA             0000000000000000  000001e0
       0000000000000018  0000000000000018   I       9     7     8
  [ 9] .symtab           SYMTAB           0000000000000000  000000e0
       00000000000000f0  0000000000000018          10     9     8
  [10] .strtab           STRTAB           0000000000000000  000001d0
       000000000000000d  0000000000000000           0     0     1
  [11] (.shstrtab         STRTAB           0000000000000000  000001f8
       0000000000000067  0000000000000000           0     0     1
~~~
-S 옵션을 통해 section의 구조를 보면 .text, .data, .bss 등의 section으로 나누어져 있는 것을 볼 수 있다.  

>readelf -l main.o

~~~
There are no program headers in this file.
~~~
하지만 Segment구조를 보는 -l 명령어는 아직 segment를 찾지 못하는데 link과정 후에 알아보자.  

다른 방법으로는 assemble하기 전으로 돌아가서 어셈블리 코드를 보는 방법도 있다. 이 과정을 disassemble이라고 한다.
> objdump -d main.o

~~~
main.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <main>:
   0:	f3 0f 1e fa          	endbr64
   4:	55                   	push   %rbp
   5:	48 89 e5             	mov    %rsp,%rbp
   8:	c7 45 f8 03 00 00 00 	movl   $0x3,-0x8(%rbp)
   f:	c7 45 fc 01 00 00 00 	movl   $0x1,-0x4(%rbp)
  16:	b8 00 00 00 00       	mov    $0x0,%eax
  1b:	5d                   	pop    %rbp
  1c:	c3                   	retq
~~~
## Link(main)
이 과정은 외부 라이브러리를 연결하는 과정이다. libc에 있는 함수를 사용하면 libc.so 파일을 연결하게 되는데 이런 과정을 ld가 담당한다. 이때 linking을 static하게하면 파일에 모든 라이브러리 함수를 저장하고 dynamic하게 하면 라이브러리 주소를 통해 함수를 불러올 수 있다.
마찬가지로 readelf로 확인하면 아래와 같다.
> readelf -l main

~~~
Elf file type is DYN (Shared object file)
Entry point 0x1040
There are 13 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000002d8 0x00000000000002d8  R      0x8
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x00000000000005c8 0x00000000000005c8  R      0x1000
  LOAD           0x0000000000001000 0x0000000000001000 0x0000000000001000
                 0x00000000000001d5 0x00000000000001d5  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x0000000000000130 0x0000000000000130  R      0x1000
  LOAD           0x0000000000002df0 0x0000000000003df0 0x0000000000003df0
                 0x0000000000000220 0x0000000000000228  RW     0x1000
  DYNAMIC        0x0000000000002e00 0x0000000000003e00 0x0000000000003e00
                 0x00000000000001c0 0x00000000000001c0  RW     0x8
  NOTE           0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000020 0x0000000000000020  R      0x8
  NOTE           0x0000000000000358 0x0000000000000358 0x0000000000000358
                 0x0000000000000044 0x0000000000000044  R      0x4
  GNU_PROPERTY   0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000020 0x0000000000000020  R      0x8
  GNU_EH_FRAME   0x0000000000002004 0x0000000000002004 0x0000000000002004
                 0x000000000000003c 0x000000000000003c  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000002df0 0x0000000000003df0 0x0000000000003df0
                 0x0000000000000210 0x0000000000000210  R      0x1

 Section to Segment mapping:
  Segment Sections...
   00
   01     .interp
   02     .interp .note.gnu.property .note.gnu.build-id .note.ABI-tag .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn
   03     .init .plt .plt.got .text .fini
   04     .rodata .eh_frame_hdr .eh_frame
   05     .init_array .fini_array .dynamic .got .data .bss
   06     .dynamic
   07     .note.gnu.property
   08     .note.gnu.build-id .note.ABI-tag
   09     .note.gnu.property
   10     .eh_frame_hdr
   11
   12     .init_array .fini_array .dynamic .got
~~~
-l 옵션을 이용하면 segment구성을 볼 수 있는데 Program Header와 Section to Segment mapping이 있는 것을 볼 수 있다.  따라서 link를 하기 전에는 section만 존재하고 link를 한 후에는 Segment가 존재한다. 이 프로그램이 메모리로 적재될 때도 이 구조가 그대로 메모리에 적재되기 때문에 메모리에 Segment가 존재하는 것이다.
