---
title: "[RE] Register & Assembly Basic"
category: Reverse Engineering
excerpt: "레지스터, 어셈블리어 기본설명"
toc: true
toc_sticky: true
toc_label: "Table of Contents"
toc_icon: "bars"
---

# 1. 레지스터
레지스터는 프로세서의 저장장치이다. 프로세서의 종류를 레지스터의 크기에 따라 32bit(x86), 64bit(x86-64)로 나눈다.   
레지스터의 종류로는 EAX, ECX, EDX, EBX / ESI, EDI / EBP, ESP, EIP / EFLAGS / CS, SS, DS, ES, FS, GS 가 있다.    
이후에 하나씩 설명하겠다.
![x86 register](https://user-images.githubusercontent.com/45323902/152172176-4395a8e8-8030-416e-bf19-1388188bcc05.png)

우선 레지스터는 크기에 따라 이름이 다르기 때문에 알아두면 쓸곳이 많다.     
예시로 AH, AL은 8bit, AX는 16bit, EAX는 32bit, RAX는 64bit의 레지스터이다. 다른 레지스터에도 똑같이 적용된다.
![Regiter Size](https://user-images.githubusercontent.com/45323902/152172148-c8bad396-7180-4a5f-8f6d-7aa1c6b87612.png)
 
## IA-32 Basic Program Execution Register
다음은 Intel Architecture 32bit 아키텍처에서 사용하는 레지스터의 이름과 간략한 설명이다. 레지스터는 크게 4종류로 나눈다.  
![General-Purpose Registers](https://user-images.githubusercontent.com/45323902/152739748-57b605af-700b-4e42-aca1-d0097ed746ac.png)  
### General-Purpose Register
1. EAX(Accumulator Register): 계산에 사용되는 레지스터이며 operand, result 모두 저장한다.
2. ECX(Counter Register): 문자열, 반복문에 사용되는 레지스터이다.
3. EDX(Data Register): I/O 포인터로 사용된다.
4. EBX(Base Register): DS에 속한 데이터의 포인터로 사용된다.
5. ESI(Source Index Register): 문자열의 source pointer로 이용된다. DS에 있는 데이터 포인터.
6. EDI(Destination Index Register): 문자열의 destination pointer로 이용된다. ES에 있는 데이터 포인터.
7. EBP(Base Pointer Register): SS에 속한 stack base pointer. stack frame의 base pointer이다.
8. ESP(Stack Pointer Register): SS에 속한 stack pointer. 유동적인 stack pointer이다. EBP와 ESP는 function call 방식에서 다시 다룬다.

각 레지스터는 기본적으로 특정 세그먼트를 기본값으로 가진다. 자세한 내용은 [Segment](https://ho7221.github.io/operating%20system/Segment/)에 있다.
{: .notice--info}
### Segment Register(16-bit)
세그먼트는 말 그대로 메모리의 조각을 말하는데, 실제 메모리 공간을 여러 부분으로 나누어 관리하는 방식이다.  
자세한 내용은 [Segment](https://ho7221.github.io/operating%20system/Segment/)을 참조하자.  
1. CS : Code Segment
2. DS : Data Segment
3. SS : Stack Segment
4. ES : Extra Segment
5. FS : F Segment (2nd ES)
6. GS : G Segment (3rd ES)

### Flag Register
![EFLAGS](https://user-images.githubusercontent.com/45323902/152793060-f78f3149-fb15-4f1b-8687-535be3fb4a7e.png)  
플래그는 여러 시스템 상태를 True, False로 나타내는 방식이다. Flag의 종류중에 Z(ero), S(ign), C(arry), O(verflow), D(irection) 정도만 보면 될것같다.  
1. EFLAGS

### Instruction Pointer
1. EIP: 다음 실행될 명령어가 있는 위치를 CS로부터의 offset으로 가진다. 직접 변경할 수 없고 JMP, CALL, RET 등의 명령어, interrupt, exception에 의해서 변경된다. 

# 2. 어셈블리
우리가 아는 c언어, python 등 영어를 조금만 알면 이해할 수 있는 직관적인 언어들은 프로세서가 이해할 수 없다. 프로세서는 미리 정해진 아키텍처에 따른 명령어만 이해할 수 있기 때문이다. 그래서 gcc와 같은 컴파일러가 소스코드를 어셈블리어로 번역해서 컴퓨터가 이해할 수 있도록 해준다.  
하지만 이 과정에서 소스코드 정보는 대부분 사라지기 때문에 역으로 코드를 보려할 때(disassemble) 어셈블리어로 된 코드밖에 볼 수 없다. 그래서 어셈블리어를 알고 소스코드의 의도를 파악할 수 있어야 한다.

물론 hey-ray의 IDA를 이용하면 소스코드도 복원할 수 있긴 하다. 하지만 어셈블리어를 공부할때는 최대한 참아보자.
{: .notice--info} 
## 어셈블리어 기본 설명
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
  
WORD는 원래 프로세서가 한번에 처리할 수 있는 데이터의 크기를 나타낸다. 하지만 Intel은 초기 16bit에서 WORD를 정의하고 바꾸지 않는다. 그래서 32bit는 Double WORD 64bit는 Quad WORD가 된 것이다.
{: .notice--info}
AT&T 방식에서 Mov에 l을 붙이는 이유는 오퍼랜드의 다룰 데이터의 크기를 나타내는 것인데, l은 long(32bit int)를 나타낸다.  
그 외에 b,s,w,q,t 등이 있다.
## 어셈블리 명령어
1. MOV(Move)
    > - MOV DEST,SRC // src를 dest에 옮긴다. 하지만 src의 데이터는 변하지 않는다.  
    > - MOV EAX,[ESP+0x10] // EAX=\*(ESP+0x10) ESP+0x10 주소에 있는 내용을 eax에 대입한다.  
    > - MOV [EAX],EBX // \*EAX=EBX EBX의 내용을 EAX주소가 가리키는 메모리에 대입한다.   
    > - MOV ES:[EBX], EAX // segment override라고 부르며 EBX의 기본 세그먼트 DS 대신 ES를 사용한다. EBX는 오프셋이며 이 주소가 가리키는 메모리에 EAX를 대입한다.  
    > - MOV BYTE PTR EBX,[ESI+ECX] // ESI+ECX주소의 내용을 BYTE의 크기만큼 EBX에 저장한다.   

    다음은 틀린 구문이다.  
    > - MOV EAX,[EAX-EBX] // 레지스터는 더하기만 가능하므로 틀린 구문이다.  
    > - MOV EAX,[ESI+ECX+EDX] // 레지스터는 두개까지만 더할 수 있다.  

2. LEA(Load Effective Address)  
    Effective Address(유효주소)는 Segment+Counter+Offset의 형태로 계산된 선형주소를 의미한다. 따라서 LEA는 EA를 계산, 대입하는 명령어이다.  
    > - LEA DEST,SRC // src의 주소를 계산해 주소를 dest에 load한다.  
    > - LEA EAX,[ESP+ECX*4+0x40] 처럼 주소를 계산하고 그 주소를 EAX에 load한다.  

3. XCHG(Exchange)  
    >xchg arg1, arg2 // 두 오퍼랜드의 값을 교환한다.  

4. LODSB(Load Byte String)  
    [DS:ESI] 내용을 Byte(8bit)만큼 메모리를 읽어와 AL에 저장. 후에 DF(Direction Flag)에 따라 ESI를 변경. 외에도 LODSW, LODSD 등이 있다.  

5. STOSB(Store Byte String)  
    AL 값을 [ES:EDI]에 저장. 이후 DF에 따라 EDI를 변경. 외에도 STOSW, STOSD 등이 있다.

6. PUSH   
    <img width="733" alt="Push Operation" src="https://user-images.githubusercontent.com/45323902/152172318-51dcc031-08fd-4ee1-864d-179fd8f0cfe0.png">   
    오퍼랜드의 값을 스택에 넣는다. 그 후 ESP를 감소시킨다. 
    
    중요한 점은 스택은 자랄수록 주소가 줄어든다. 그래서 스택은 아래로 자란다(?). 절대 그림을 그리면서 헷갈려하지 말자.
    {: .notice--danger}

7. POP
    <img width="881" alt="Before Popping Doubleword" src="https://user-images.githubusercontent.com/45323902/153809774-64b0029e-e79e-4f11-904e-9140674d6c4a.png">
    PUSH와 반대로 스택의 최상단의 값을 오퍼랜드에 대입한다. ESP는 증가한다.  

8. Jxx
    분기명령으로 Jmp, Jne, Jbe 등 Jxx 이전의 비교구문에 따른 ZF, CF 등 플래그에 따라 특정 분기로 이동한다. 

9. CALL, RET
    CALL은 특정 주소나 프로시저로 EIP를 이동시키며 돌아올 주소를 스택에 PUSH한다. RET는 POP EIP를 통해 EIP를 복원시킨다.

