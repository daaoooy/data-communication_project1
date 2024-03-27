_# Data Communication (202110789 유다연)
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

### 1. Sender Program


### 2. Receiver Program
Receiver program은 Sender로부터 파일을 받고 데이터를 저장한다.

포트 넘버와 버퍼 사이즈는 아래와 같이 정의해주었다.

    #define PORT 12345
    #define BUFFER_SIZE 1024

      
사용한 변수 및 버퍼는 아래와 같다.

    int sockfd, n;
    struct sockaddr_in server_addr, client_addr;
    socklen_t client_size;
    
    char buffer[BUFFER_SIZE];
    char file_name[BUFFER_SIZE];
    char save_file_name[BUFFER_SIZE];

- 설명
  > sockfd: 소켓의 파일 디스크립터
  > 

##### 우선 socket () 함수를 통해 소켓을 생성해주고 주소를 설정해주었다. 
##### socket(), memset(), bind() 함수를 사용해 주었다
- socket() : 사용하고자 하는 통신 프로토콜을 지정해준다.
- bind() : 서버에서 소켓에 이름을 부여한다.
- memset() 바이트 영역을 특정 값으로 설정한다.
  

      if((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
              perror("Sockent creation failed.");
              exit(EXIT_FAILURE);
      }
      
      server_size = sizeof(server_addr);
      client_size = sizeof(client_addr);
  
      memset(&server_addr, 0, server_size);
      server_addr.sin_family = AF_INET;
  
      server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
      server_addr.sin_port = htons(PORT);
  
      if(bind(sockfd, (struct sockaddr *)&server_addr, server_size) < 0) {
              perror("bind failed.");
              exit(EXIT_FAILURE);
      }



      
