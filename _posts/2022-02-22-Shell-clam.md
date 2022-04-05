---
title : "[RE]Shell Clam"
categories : CTF
tags : [Reversing]
excerpt : "Dreamhack Shell Clam"
toc : true
toc_label : "Table of Contents"
toc_sticky : true
toc_icon : bars
---

# 코드 확인
## 1. server.py
main함수는 입력받은 값과 임의로 10-99범위의 정수 10~20개를 랜덤으로 만든 prob를 runner에 전달한다. 그리고 랜덤으로 만든 prob를 통해 answer라는 값을 만들고 runner의 return값과 비교한 후 맞으면 flag파일을 읽어온다.   
## 2. runner.c  
hex2bin 함수는 입력의 타입 string을 byte형태로 바꾸어주는 역할을 한다. 이 함수가 없다면 따로 이 기능을 하는 함수를 만들어주어야 하니 친절하다.  
main 함수는 argv[1]에 shellcode, argv[2]부터 server.py의 prob를 입력으로 받는다. 이후 shellcode를 hex2bin으로 형태를 바꾸고 foo라는 함수의 프로토타입을 만들어준다. 이 함수는 int, char\*[]를 입력으로 받는데 각각 argc, argv를 의미한다.   
이후 shellcode의 주소를 호출한다. 따라서 shellcode에 입력된 값을 실행하게 될 것이다.   
하지만 seccomp가 걸려있으니 이후에 Syscall은 이용할 수 없다. 즉 execve와 같은 방식으로 할 수 없고 라이브러리 함수도 호출할 수 없다.  
만약 제대로 shellcode가 호출되면 shellcode 함수의 return값이 return 될것이고 seccomp에 걸리는 등의 오류가 생긴다면 음수의 return값이 나올것이다.

만약 -11이 나온다면 SegFault오류이니 seccomp에 위반되는 함수를 사용했는지 확인하자. 
{: .notice--info}
## 3. Shellcode
~~~
int foo(int argc, char* argv[]){
    int answer=0, s=0;
    while(argv[++s]!=NULL);
    s-=2;
    for(int i=0;i<s;i++){
        int n=(argv[i+2][0]-'0')*10+argv[i+2][1]-'0';
        if(n%3) answer+=n*2;
        else answer+=n;
    }
    answer%=100;
    return answer;
}
~~~
이 코드를 컴파일하고 hex값을 hxd로 추출해서 넣으면 flag가 나온다.  
<img width="1502" alt="flag" src="https://user-images.githubusercontent.com/45323902/155170852-6b7b8365-f573-4458-9766-a0d3b7962cdf.png">
