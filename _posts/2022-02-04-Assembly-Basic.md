---
layout: single
title: "02. Assembly Basic"
subtitle: "바이너리 분석"
category: Reverse Engineering
toc: true
toc_sticky : true
toc_label : "POSTING"
toc_icon: "blog"
---

# 1. 레지스터
---
레지스터는 프로세서의 저장장치. 레지스터의 크기에 따라 32bit(x86), 64bit(x86-64)로 나눈다.   
레지스터에는 EAX, ECX, EDX, EBX / ESI, EDI / EBP, ESP, EIP / EFLAGS / CS, SS, DS, ES, FS, GS 가 있다.  
![x86 register](https://user-images.githubusercontent.com/45323902/152172176-4395a8e8-8030-416e-bf19-1388188bcc05.png)

레지스터의 크기에 따라 이름이 다르다!   
예시로 AH, AL은 8bit, AX는 16bit, EAX는 32bit, RAX는 64bit의 레지스터이다.
![Regiter Size](https://user-images.githubusercontent.com/45323902/152172148-c8bad396-7180-4a5f-8f6d-7aa1c6b87612.png)

## IA-32 Basic Program Execution Register
---
### General-Purpose Register
1. AX : Accumulator Register
2. CX : Counter Register
3. DX : Data Register
4. BX : Base Register
5. SI : Source Index Register
6. DI : Destination Index Register
7. BP : Base Pointer Register
8. SP : Stack Pointer Register

### Segment Register(16-bit)
1. CS : Code Segment
2. DS : Data Segment
3. SS : Stack Segment
4. ES : Extra Segment
5. FS : F Segment (2nd ES)
6. GS : G Segment (3rd ES)

### Flag Register
1. FLAGS

### Instruction Pointer
1. IP

# 2. 어셈블리
---
어셈블리 설명 추가

## 어셈블리어 기본 설명
---
어셈블리어는 AT&T 방식과 Intel 방식이 있다. 주로 Intel 방식을 이용한다.

Example 1. 값 대입   
>Intel : mov eax, 5   
>AT&T : mov $5, %eax

Intel 방식은 자연스럽게 해석하면 되고 오른쪽을 왼쪽으로 대입한다.  
AT&T 방식은 레지스터는 %, 정수값은 $ 를 붙여서 나타낸다. 또한 왼쪽을 오른쪽에 대입한다. 

Example 2. 메모리 참조  
>Intel : mov eax, [ebx]  
>AT&T : mov (%ebx), %eax

이 방식은 ebx가 0x12345678이면 그 주소값 안에 있는 값을 불러오는 방식이다.  
Intel 방식은 레지스터에 []를 붙여 메모리 주소를 참조한다.   
AT&T 방식은 ()를 붙여 메모리 주소를 참조한다. 

Example 3. 메모리 변위(displacement)  
>Intel : mov eax, [ebx+4]   
>AT&T : movl 4(%ebx), eax

의미는 ebx+4 주소에 있는 값을 참조해 eax에 대입이다. 이때 4를 변위라고 한다.  
Intel 방식은 직관적이고, AT&T 방식은 앞에 변위를 붙여서 나타낸다.

Example 4.   
>Intel : mov eax, [ebx+8+edi*4]   
>AT&T : movl 8(%ebx,%edi,4), %eax
 
의미는 ebx+8+edi*4를 참조해 eax에 대입이다.  
이떄 edi는 오프셋 레지스터, 4는 스칼라이다. 

Example 5. Size 표기  
>Intel : mov BYTE PTR [eax], 5   
>AT&T : movb $5, (%eax)  

Intel은 오퍼랜드가 레지스터일 경우에는 자동으로 크기를 인식하지만 메모리 주소일때는 크기를 지정해줘야 한다(SIZE PTR).  
BYTE(8bit), WORD(16bit), DWORD(32bit), QWORD(64bit) 로 구분된다.  
AT&T 방식에서 Mov에 l을 붙이는 이유는 오퍼랜드의 다룰 데이터의 크기를 나타내는 것인데, l은 long(32bit int)를 나타낸다.  
그 외에 b,s,w,q,t 등이 있다.

## 어셈블리 명령어
---
1. MOV(Move)
    >mov dest, src
    
    src를 dest에 옮긴다. 하지만 src의 데이터는 변하지 않는다.
2. LEA(Load Effective Address)  
    >lea dest, src
    
    src의 주소를 계산해 주소를 dest에 load한다. 
    lea eax, [esp + 0x40] 처럼 주소를 계산하고 그 주소를 eax에 load한다. 
3. XCHG(Exchange)  
    >xchg arg1, arg2
    
    두 오퍼랜드의 값을 교환한다.
4. LODSB(Load Byte String)
    [DS:ESI] 내용을 Byte(8bit)만큼 메모리를 읽어와 AL에 저장. 후에 DF(Direction Flag)에 따라 ESI를 변경.
       외에도 LODSW, LODSD 등이 있다. 
5. STOSB(Store Byte String)
   AL 값을 [ES:EDI]에 저장. 이후 DF에 따라 EFI를 변경. 외에도 STOSW, STOSD 등이 있다.
6. PUSH   
   <img width="733" alt="Push Operation" src="https://user-images.githubusercontent.com/45323902/152172318-51dcc031-08fd-4ee1-864d-179fd8f0cfe0.png">   
	오퍼랜드의 값을 스택에 넣는다.
