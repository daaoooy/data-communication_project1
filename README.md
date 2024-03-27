# Data Communication (202110789 유다연)
## Term Project 01
### part 01 
#### sender and receiver programs that communicate via UDP sockets
#### : UDP 소켓을 이용한 Sender, Receiver 프로그램 만들기

---
* 과제 설명
  1.  C / C++로 작성되어야 하고 리눅스 환경에서 작동해야 한다.
  2.  Sender Program 은 파일을 전송하는 역할을, Receiver Program은 데이터를 받아 파일에 저장하는 역할을 한다.
  3.  파일을 보내기 전, Sender은 "Greeting" 이라는 문장과 파일 이름을 보내야 한다.
  4.  Receiver은 3단계에서 Sender가 보낸 것들을 받으면 "OK" 라고 응답한다.
  5.  Sender는 "OK" 응답을 받으면 파일을 보내기 시작한다.
  6.  모든 데이터 전송이 완료되면 Receiver에게 "Finish"라는 문장을 보낸다.
  7.  Receiver은 파일과 "Finish"를 받으면 "WellDone"라고 응답한다.
 
---

**Ubuntu 리눅스 환경에서 C언어를 사용해 작성하였다.**

- 리눅스 컴파일
  
      gcc sender.c -o sender
      gcc receiver.c -o receiver


- 리눅스에서 작성한 프로그램 실행
  
      ./sender
      ./receiver
  
---

### 1. 프로그램 설명
+ 중간 중간 sleep(3); 을 적어놓은 이유 : Sender와 Receiver가 서로 주고 받고 하는 과정을 좀 더 자세히 보기 위해서이다.

사용한 라이브러리와 정의해준 것들은 아래와 같다.

    #include <stdio.h>
    #include <sys/socket.h>
    #include <stdlib.h>
    #include <arpa/inet.h>
    #include <string.h>
    #include <unistd.h>
    
    #define PORT 12345
    #define BUFFER_SIZE 1024
    #define IP "127.0.0.1"

#### 소켓 

소켓 생성 시 사용한 함수들이다.

    int socket(int domain, int type, int protocol);
    void* memset(void* ptr, int value, size_t num); 
    int bind(int sockfd, (struct sockaddr *) my_addr, socklen_t addrlen)

1. socket()
   - **domain** 어떤 영역에서 통신을 할지
   - **type**은 어떤 프로토콜을 사용할지 (UDP를 사용하기 위해 SOCK_DGRAM을 사용)
   - **protocol**은 도메인과 유형에 따라서 사용할 프로토콜을 결정하는 부분이다.
2. memset ()
    - **ptr** 세팅하고자 하는 메모리의 시작 주소
    - **value** 메모리에 세팅하고자 하는 값
    - **num** 길이
3. bind ()
   - 소켓에 주소를 할당 해주는 함수. ip 주소, port번호를 할당해줄 수 있음.
   - **sockfd** 소켓 디스크립터
   - **my_addr** 주소 정보를 할당.
   - **addr_len** my_addr의 길이


#### 시작    

먼저 buffer에 Greeting 이라는 문자열을 담아 보내준다. 이 때, sendto() 함수를 사용하였다.

    int sendto(int s, const void *msg, size_t len, int flags, const struct sockaddr *addr, socklen_t addr_len);

- **s**        소켓 디스크립터
- **msg**      전송할 데이터
- **len**      데이터의 바이트 단위 길이
- **flags**    전송을 위한 옵션
- **addr**     목적지 주소 정보
- **addr_len** 목적지 주소

실패 시 반환 되는 값이 -1이 이므로 0보다 작을 시 에러가 났음을 표시해주었다.

먼저 전송할 파일은 **입력**을 통해 받았다. “Enter the file name to send: " 라는 문장이 나오면 전송하고자 하는 파일 이름을 적어주었다.
예시로 “test.txt”를 입력해주었다. 파일 이름을 buffer에 입력 받아 sendto()를 이용해 receiver에게 보낸다. 에러 처리도 Greeting을 보낼 때와 같이 처리해주었다.

여기서 현재 buffer에 있을 파일 이름을 file_name에 복사해준다. 이것은 나중에 파일을 열 때 인수로 사용할 것이다.
Sender가 Receiver에게 “Greeting”과 파일 이름을 보내었다. 그러면 Receiver는 recvfrom() 함수를 통해 Sender가 보낸 내용들을 받는다.

    int recvfrom(int s, void *buf, size_t len, int flags, struct sockaddr *addr, socklen_t *addr_len)
    
- **s** 소켓 디스크립터
- **buf** 버퍼
- **len** 버퍼의 바이트 단위 길이
- **flags** 수신을 위한 옵션
- **addr** 전송한 곳의 주소 정보
- **addr_len** 전송한 주소의 정보의 크기
  
여기서도 역시 반환값이 -1이면 실패한 것으로, 에러처리를 동일하게 넣어주었다.

“Greeting”과 파일 이름을 받았다면, Receiver는 다 받았다고 응답하는 “OK”를 보내준다. 응답을 보내는 것은 “Greeting” 전송 과정과 동일하게 작성하였다. (sendto() 함수 활용)

Sender는 Receiver가 보낸 응답을 recvfrom() 함수를 통해 확인한다

이제 파일 전송을 시작할 것이다. 먼저 fopen()함수를 이용해 파일을 열어줄 것이다. 아까 파일 이름을 담아두었던 file_name을 이용해 열어주고 file에 저장한다. 

while문을 이용해 데이터를 전송한다(바이트 단위로 읽어 전송). 파일 전송을 모두 마치면 sendto() 함수를 이용해 끝났음을 알리는 “Finish” 를 보내준다.

Receiver는 파일 데이터를 전송 받아 새로운 파일에 저장할 것인데, 여기서 새로운 파일 이름(save_file_name)을 입력 받아 사용할 것이다. 추가적으로 파일 내용을 확인해보는 부분도 살짝 넣어주었다. 

새로 입력받은 파일이름을 이용하여 fopen() 함수를 통해 생성해주고 그 파일에 전송 받은 데이터를 저장해준다. fputs 함수를 사용하였고 이 생성된 파일은 같은 디렉토리에서 확인해볼 수 있다.

Receiver는 Sender에게 Finish 응답을 받고, 데이터 저장까지 마쳤으면 Sender에게 잘 받았다는 “WellDone”을 응답으로 보내준다. Sender는 Receiver가 보낸 응답을 확인하고, 프로그램은 종료가 된다.

### 2. 프로그램 실행 과정

1. 실행 했을 때의 모습

![스크린샷 2024-03-27 233520](https://github.com/daaoooy/data-communication_project1/assets/143688136/0ecfab62-f5af-4042-92d9-d776914e09df)

2. 파일 이름을 입력 받고 Receiver가 "OK"라고 응답, 응답을 받은 Sender은 파일을 전송하기 시작한다.

![스크린샷 2024-03-27 233543](https://github.com/daaoooy/data-communication_project1/assets/143688136/ca62dd2e-f75e-4ff1-b625-bcd4d6d1f6e1)

3. 파일 전송 및 finish 응답 전송, 응답 확인
   
![스크린샷 2024-03-27 233554](https://github.com/daaoooy/data-communication_project1/assets/143688136/d7ccd10e-088f-4861-8995-694dc5a8e41b)

![image](https://github.com/daaoooy/data-communication_project1/assets/143688136/67ca2047-44f4-47c0-b21c-4c422284de98)


4. 최종 출력 모습
![image](https://github.com/daaoooy/data-communication_project1/assets/143688136/0217d1e7-9b40-407f-a78d-be73bef5f8f8)

과정 사진에서는 모두 담기지 않았지만, 실행해보면 sleep을 통해 과정이 한 단계씩 화면에 출력됨을 볼 수 있다. 
   
### 3. 소스코드
소스코드는 README 파일과 함께 따로 .c 파일로 제출하였음.

### (+) 수정 보안
- 이것은 현재 과제 제출용으로, 앞으로 나올 part 들과 또 다른 Project 과제를 위해 수정 보완이 있을 것이다.

      
