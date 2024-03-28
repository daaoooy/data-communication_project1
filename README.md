# Data Communication
## Term Project 01
### part 01
#### sender and receiver programs that communicate via UDP sockets
#### : UDP 소켓을 이용한 Sender, Receiver 프로그램 만들기

---

* **과제 설명**
  
  1.  C / C++로 작성되어야 하고 리눅스 환경에서 작동해야 한다.
  2.  Sender Program 은 파일을 전송하는 역할을, Receiver Program은 데이터를 받아 파일에 저장하는 역할을 한다.
  3.  파일을 보내기 전, Sender은 "Greeting" 이라는 문장과 파일 이름을 보내야 한다.
  4.  Receiver은 3단계에서 Sender가 보낸 것들을 받으면 "OK" 라고 응답한다.
  5.  Sender는 "OK" 응답을 받으면 파일을 보내기 시작한다.
  6.  모든 데이터 전송이 완료되면 Receiver에게 "Finish"라는 문장을 보낸다.
  7.  Receiver은 파일과 "Finish"를 받으면 "WellDone"라고 응답한다.
 
---

* **개발 환경**
  
  Ubuntu 리눅스 환경에서 C언어를 사용해 작성하였다.
  
   [리눅스 컴파일]
    
        gcc sender.c -o sender
        gcc receiver.c -o receiver
  
  
   [리눅스에서 작성한 프로그램 실행]
    
        ./sender
        ./receiver

 
---

  * **헤더 파일**
    
     사용한 헤더들이다. 기본적인 함수들과 소켓 생성, 파일에 관한 함수와 나머지 추가적인 부분(문자열 복사, 에러 처리 등)에 사용하기 위해서  아래와 같은 헤더들을 선언해주었다.
    
        #include <stdio.h>
        #include <sys/socket.h>
        #include <stdlib.h>
        #include <arpa/inet.h>
        #include <string.h>
        #include <unistd.h>
 
---

## 1. Sender Program

---

## 2. Receiver Program



