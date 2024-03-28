# Data Communication
## Term Project 01
### part 01
sender and receiver programs that communicate via UDP sockets

UDP 소켓을 이용한 Sender, Receiver 프로그램 만들기

---

### **과제 설명**
  
  1.  C / C++로 작성되어야 하고 리눅스 환경에서 작동해야 한다.
  2.  Sender Program 은 파일을 전송하는 역할을, Receiver Program은 데이터를 받아 파일에 저장하는 역할을 한다.
  3.  파일을 보내기 전, Sender은 "Greeting" 이라는 문장과 파일 이름을 보내야 한다.
  4.  Receiver은 3단계에서 Sender가 보낸 것들을 받으면 "OK" 라고 응답한다.
  5.  Sender는 "OK" 응답을 받으면 파일을 보내기 시작한다.
  6.  모든 데이터 전송이 완료되면 Receiver에게 "Finish"라는 문장을 보낸다.
  7.  Receiver은 파일과 "Finish"를 받으면 "WellDone"라고 응답한다.
 
---

### **개발 환경**
  
  Ubuntu 리눅스 환경에서 C언어를 사용해 작성하였다.
  
   [리눅스 컴파일]
    
      gcc sender.c -o sender
      gcc receiver.c -o receiver
  
  
   [리눅스에서 작성한 프로그램 실행]
    
      ./sender
      ./receiver

 
---

  ### **헤더 파일**
    
  사용한 헤더들이다. 기본적인 함수들과 소켓 생성, 파일에 관한 함수와 나머지 추가적인 부분(문자열 복사, 에러 처리 등)에 사용하기 위해서  아래와 같은 헤더들을 선언해주었다.

    
      #include <stdio.h>
      #include <sys/socket.h>
      #include <stdlib.h>
      #include <arpa/inet.h>
      #include <string.h>
      #include <unistd.h>
 
---

  ### **사용한 변수 및 define**
  
  포트 번호로는 12345, 버퍼 사이즈는 1024, IP는 "127.0.0.1"을 사용해주었다. 지정해준 IP는 컴퓨터의 네트워크 기능을 시험하기 위해 가상으로 인터넷망에 연결되어 있는 것처럼 할당하는 인터넷 주소이다.
  
      #define PORT 12345                                                                                    
      #define BUFFER_SIZE 1024                                                                              
      #define IP "127.0.0.1"
  
  1. **int sockfd** : 소켓 디스크립터로 사용한다.
  2. **int n** : recvfrom 함수가 반환하는 값을 받을 것이다.
  3. **int sender_size, receiver_size** : sizeof() 를 이용해 주소들의 사이즈를 담을 것이다.
  4. **char buffer [BUFFER_SIZE]** : 전송할 내용을 담아 사용할 것이다. (전송할 때 마다 재사용)
  5. **char file_name [BUFFER_SIZE]** : 버퍼는 계속해서 담는 내용들이 바뀌므로, file_name에 입력 받은 파일 이름을 담아 놓는다.
  6. **struct sockaddr_in receiver_addr, sender_addr** : 소켓의 주소를 담는다. (구조체)
  7. **save_file_name[BUFFER_SIZE]** : Receiver에서만 사용하는 변수. 데이터를 받아 저장할 파일 이름을 저장할 것이다.

(+)

6번 socketaddr_in 구조체에 대해 간단하게 설명하자면,

먼저 socketaddr 구조체는 2개의 멤버 변수를 가진다.
1. sa_family : 주소 체계를 구분하기 위한 변수 
2. sa_data : 실제 주소를 저장하기 위한 변수

이 프로그램에서 사용한 socketaddr_in 은 위 구조체에서 sa_family가 AF_INET 인 경우에 사용하는 구조체이다.
사용하는 IP주소는 IPv4 주소 체계이다. 
1. **sin_family** : 항상 AF_INET으로 설정해준다.
2. **sin_port** : 포트 번호를 설정해준다.
3. **sin_addr** : 호스트의 ip 주소
4. **sin_zero** : 반드시 모두 0으로 채워져 있어야 하는 8비트의 dummy 데이터이다.
 
---

  ### **사용한 함수 (중요하게 사용된 것들)**
  
  > int socket(int domain, int type, int protocol);

   - 사용하고자 하는 총신 프로토콜을 지정한다.
   - **domain** 어떤 영역에서 통신을 할지
   - **type**은 어떤 프로토콜을 사용할지 (UDP를 사용하기 위해 SOCK_DGRAM을 사용)
   - **protocol**은 도메인과 유형에 따라서 사용할 프로토콜을 결정하는 부분이다.

     
  > void* memset(void* ptr, int value, size_t num);

   - 메모리를 특정 값으로 세팅한다.  
   - **ptr** 세팅하고자 하는 메모리의 시작 주소
   - **value** 메모리에 세팅하고자 하는 값
   - **num** 길이


  > int bind(int sockfd, (struct sockaddr *) my_addr, socklen_t addrlen)
     
   - 소켓에 주소를 할당 해주는 함수.
   - **sockfd** 소켓 디스크립터
   - **my_addr** 주소 정보를 할당.
   - **addr_len** my_addr의 길이

  > int sendto(int s, const void *msg, size_t len, int flags, const struct sockaddr *addr, socklen_t addr_len);

  - 데이터를 UDP 패킷으로 상대방에게 전송한다.
  - **s**        소켓 디스크립터
  - **msg**      전송할 데이터
  - **len**      데이터의 바이트 단위 길이
  - **flags**    전송을 위한 옵션
  - **addr**     목적지 주소 정보
  - **addr_len** 목적지 주소

  > int recvfrom(int s, void *buf, size_t len, int flags, struct sockaddr *addr, socklen_t *addr_len)

  - 데이터를 UDP 패킷으러 상대방으로부터 받는다.
  - **s** 소켓 디스크립터
  - **buf** 버퍼
  - **len** 버퍼의 바이트 단위 길이
  - **flags** 수신을 위한 옵션
  - **addr** 전송한 곳의 주소 정보
  - **addr_len** 전송한 주소의 정보의 크기

---


# 1. Sender Program  

### - 프로그램 코드 설명

#### 소켓 생성 및 주소 설정 
    if ((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
    	perror("Socket Creation failed.");
    	exit(EXIT_FAILURE);
    }

    sender_size = sizeof(sender_addr);
    receiver_size = sizeof(receiver_addr);  
    
    memset(&receiver_addr, 0, receiver_size);
    receiver_addr.sin_family = AF_INET;
    
    inet_pton(AF_INET, IP, &(receiver_addr.sin_addr.s_addr));
    receiver_addr.sin_port = htons(PORT);


먼저 소켓을 생성해준다. socket() 함수를 사용해주었고 IPv4 영역에서 통신하기 위해 AF_INET을, UDP 통신을 위해 SOCK_DGRAM을, 프로토콜 값으론 0을 사용했다. 소켓 생성에 실패했을 때를 대비하여 에러 처리도 해주었다. memset()으로 메모리의 내용을 원하는 크기만큼 특정 값으로 세팅한다. 앞에서 sockaddr_in 구조체를 설명했듯이 sin_family는 AF_INET으로 해준다.

여기서 inet_pton() 함수를 사용해 문자열 형태의 ip 주소를 binary 형태로 변환 후 receiver_addr 구조체의 호스트 ip 주소를 지정해주었고, htons() 함수를 사용해 PORT (12345로 지정해주었음)를 호스트 바이트 순서에서 네트워크 바이트 순서로 바꾸어주어 receiver_addr의 sin_port를 지정해주었다.

---

#### Greeting 보내기

    strcpy(buffer, "Greeting");
    if (sendto(sockfd, buffer, strlen(buffer), 0, (struct sockaddr *)&receiver_addr, receiver_size) < 0) {
    	perror("sendto failed.");
    	exit(EXIT_FAILURE);
    }

먼저 buffer에 Greeting이라는 문자열을 담아주었다. (strcpy 함수 사용) buffer에 담긴 Greeting을 sendto()함수를 이용해 receiver에게 보내준다. sendto() 안에는 소켓 디스크립터, 버퍼 (전송할 내용), 버퍼의 길이, 목적지 주소 정보와 목적지 주소가 인수로 들어간다. 주고 받고 하는 과정은 같은 작업이 반복되므로 인수 설명은 추가적인 것만 설명하도록 하겠다. 

실패 시 반환 되는 값이 -1이 이므로 0보다 작을 시 에러가 났음을 표시해주었다.

---

#### 파일 이름 보내기
    printf(">> Enter the file name to send: ");
    fgets(buffer, BUFFER_SIZE, stdin);
    
    if (sendto(sockfd, buffer, strlen(buffer), 0, (struct sockaddr *)&receiver_addr, receiver_size) < 0) {
    	perror("sendto failed.");
    	exit(EXIT_FAILURE);
    }
    
    strcpy(file_name, buffer);
    file_name[strlen(file_name)-1] = '\0';

전송할 파일의 이름을 입력을 통해 받았다. “Enter the file name to send: " 라는 문장이 나오면 전송하고자 하는 파일 이름을 적어주었다. 파일 이름을 buffer에 입력 받아 sendto()를 이용해 receiver에게 보낸다. 에러 처리도 Greeting을 보낼 때와 같이 처리해주었다. 나중에 파일을 열 때 인수로 넣을 파일 이름이 필요하기 때문에 file_name에 넣어주었다.

---

#### OK 응답 받기

  	n = recvfrom(sockfd, buffer, BUFFER_SIZE, 0, (struct sockaddr *)&sender_addr, &sender_size);
  	recvfrom_check(n);
    buffer[n] = '\0';	
  	printf("Receiver Response > %s\n", buffer);

Receiver은 Greeting과 파일 이름을 잘 받았다면 OK 라는 응답을 보내줄 것이다. Sender은 이를 recvfrom() 함수를 통해 확인할 수 있다. 응답을 확인하기 위해 buffer에 담긴 OK를 화면에 출력해주었다.

여기서 recvfrom_check()는 에러 확인을 해주는 함수인데 프로그램 작성에 있어 자주 반복되어 따로 함수로 정의해주었다.

---


#### 파일 전송하기

    FILE *file = fopen(file_name, "r");
    	
    if (file == NULL) {
    	perror("fopen failed");
    	exit(EXIT_FAILURE);
    }
    
    printf("--- Sender is sending the File ---\n");
    
    while(!feof(file)) {
    	size_t bytes_read = fread(buffer, 1, BUFFER_SIZE, file);
    	if (sendto(sockfd, buffer, bytes_read, 0, (const struct sockaddr *)&receiver_addr, receiver_size) < 0) {
    		perror("sendto failed");
    		exit(EXIT_FAILURE);
    	}
    }

    fclose(file);

Receiver에게 응답도 받았으니 이제 파일을 전송해줄 것이다. 파일은 기본적으로 fopen()함수를 이용해 열어주었고 첫 번째 인수는 파일 이름이, 두 번째 인수에는 권한을 적어준다. 아까 파일 이름을 담아둔 변수 file_name을 사용한다. 먼저 파일 오픈에 있어 실패할 경우를 대비해 에러 처리를 해주었다. 화면에 전송 중임을 표시하는 문장도 출력해주었다.

while문을 이용해 데이터를 전송한다. 바이트 단위로 읽어 전송해주었다. 이 역시도 sendto() 함수를 이용해 receiver에게 전송한다.

---


#### 파일 전송 마치면 Finish 응답 보내기

    printf("Sending \"finish\"..\n");
    
    strcpy(buffer, "Finish");
    if (sendto(sockfd, buffer, strlen(buffer), 0, (struct sockaddr*)&receiver_addr, receiver_size) < 0) {
    	perror("sendto failed.");
    	exit(EXIT_FAILURE);
    }

파일 전송을 마치면 "Greeting" 을 전송했던 것과 동일한 방식으로 Receiver에게 "Finish"를 보내준다.

---

#### WellDone 응답 받기

  	n = recvfrom(sockfd, buffer, BUFFER_SIZE, 0, (struct sockaddr *)&sender_addr, &sender_size);
  	recvfrom_check(n);
  	buffer[n] = '\0';
  	printf("Receiver Response > %s \n\n", buffer);

파일 전송도 완료하고 Finish 응답도 보냈다면 Receiver은 Sender에게 잘 받았다는 WellDone을 보내줄 것이다. 이를 recvfrom을 통해 받고 확인을 위해 화면에 출력해준다.


---






# 2. Receiver Program

#### 소켓 생성 및 주소 설정 + binding

    if((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
    	perror("Socket creation failed.");
    	exit(EXIT_FAILURE);
    }	

    receiver_size = sizeof(receiver_addr);
    sender_size = sizeof(sender_addr);
    
    memset(&receiver_addr, 0, receiver_size);
    receiver_addr.sin_family = AF_INET;
    
    receiver_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    receiver_addr.sin_port = htons(PORT);
    
    if(bind(sockfd, (struct sockaddr *)&receiver_addr, receiver_size) < 0) {
    	perror("bind failed.");
    	exit(EXIT_FAILURE);
    }

소켓 생성과 주소 설정 부분은 Sender에서 설명한 것과 동일한데 Receiver에는 bind를 하는 부분이 있다. bind는 서버에서 소켓에 주소를 할당해준다. 

---

#### Greeting 받기

  	n = recvfrom(sockfd, buffer, BUFFER_SIZE, 0, (struct sockaddr *)&sender_addr, &sender_size);
  	recvfrom_check(n);
  	buffer[n] = '\0';
  	
  	printf("\"%s\" received successfully.\n", buffer);

Sender가 보낸 "Greeting"을 받기 위해 recvfrom() 으로 데이터를 받아준다. 에러가 뜨진 않았는지 체크해주고 성공적으로 받았음을 알리는 문장을 출력해준다. 

---


#### 파일 이름  받기

  	n = recvfrom(sockfd, buffer, BUFFER_SIZE, 0, (struct sockaddr *)&sender_addr, &sender_size);
  	recvfrom_check(n);
  	buffer[n] = '\0';
  
  	strcpy(file_name, buffer);
  	file_name[strlen(file_name)-1] = '\0';
  
  	printf("\"file name(%s)\" received successfully.\n", file_name);

파일 이름도 역시 위와 동일한 방식으로 받아준다. Sender은 "Greeting"을 보낸 후 파일 이름도 보내줄 것이다. 파일 이름을 사용할 수도 있을 경우를 대비해 file_name 변수에 이름을 담아주었다. 

파일 이름을 잘 전송 받았다는 것을 알리는 문장을 출력해준다.

---


#### OK 응답 보내기

  	printf("Sending \"OK\"..\n");
  
  	strcpy(buffer, "OK");
  	if (sendto(sockfd, buffer, strlen(buffer), 0, (struct sockaddr *)&sender_addr, sender_size) < 0) {
  		perror("sendto failed.");
  		exit(EXIT_FAILURE);
  	}
	
Greeting과 파일 이름을 잘 받았다면 Receiver는 Sender에게 잘 받았다는 응답 "OK"를 보낸다. sendto()를 사용해주어 데이터를 전송한다. 

---


#### 파일 데이터 받기

  	printf("---- ..Receiving File.. ----\n");
   
  	n = recvfrom(sockfd, buffer, BUFFER_SIZE, 0, (struct sockaddr *)&sender_addr, &sender_size);
  	recvfrom_check(n);
  	buffer[n] = '\0';

OK 응답을 받은 Sender는 Receiver에게 파일을 보내기 시작할 것이다. Receiver는 recvfrom()을 사용해 파일을 전송 받는다.

---


#### 받은 데이터 저장할 파일 이름 입력 받고 파일에 데이터 저장하기

  	printf(">> Enter the file name to save: ");
  	fgets(save_file_name, BUFFER_SIZE, stdin);
  	save_file_name[strlen(save_file_name)-1] = '\0';
   
        FILE *new_file = fopen(save_file_name, "w");
        fputs(buffer, new_file);

 	fclose(new_file);

Sender로 부터 받은 파일 데이터를 새로운 파일로 저장해줄 것인데, 새로운 파일의 이름은 입력으로 받는다. fgets()로 파일 이름을 입력 받고,  fopen()으로 새로운 파일을 생성 후  fputs()로 그 파일에 데이터를 적는다.

---


#### Finish 응답 받기

  	n = recvfrom(sockfd, buffer, BUFFER_SIZE, 0, (struct sockaddr *)&sender_addr, &sender_size);
  	recvfrom_check(n);
  	buffer[n] = '\0';
  
  	printf("Sender Response > %s\n", buffer);

Receiver는 Sender가 파일 전송을 마쳤음을 알리는 Finish 응답을 받는다. 화면에 응답을 잘 받았다는 출력을 보내준다.

---


#### WellDone 응답 보내기

    printf("Sending \"WellDone\"..\n\n");
    	 
    strcpy(buffer, "WellDone");
    if (sendto(sockfd, buffer, strlen(buffer), 0, (struct sockaddr *)&sender_addr, sender_size) < 0) {
    	perror("sendto failed");
    	exit(EXIT_FAILURE);
    }

파일을 잘 전송 받았고, Finish 응답까지 받았다면 Receiver는 Sender에게 sendto() 함수를 이용해 잘 받았다고 WellDone 응답을 보내준다. 

---


# 프로그램 실행 화면


1. 프로그램 실행시 첫 화면
![스크린샷 2024-03-28 233811](https://github.com/daaoooy/data-communication_project1/assets/143688136/5a6ec0f5-9214-4be3-ab59-ccae30582439)

2. 파일 입력 후 OK 응답을 주고 받는 모습
![스크린샷 2024-03-28 233826](https://github.com/daaoooy/data-communication_project1/assets/143688136/f04969e7-24db-4960-bc81-60f63787a86b)

3. 파일을 보내기 시작
![스크린샷 2024-03-28 231341](https://github.com/daaoooy/data-communication_project1/assets/143688136/886716fe-4b1e-498c-9275-b69359f3889b)

4. 새로운 파일 저장 후 Finish 응답까지 받은 상태
![스크린샷 2024-03-28 231415](https://github.com/daaoooy/data-communication_project1/assets/143688136/01bca504-1770-4a48-b925-7c0958fae934)

5. WellDone 응답까지 완료한 후 최종 모습
![스크린샷 2024-03-28 232305](https://github.com/daaoooy/data-communication_project1/assets/143688136/752eb4b5-9d20-4c89-8cdb-a294a51bcca1)

6. 데이터를 저장한 새로운 파일
![스크린샷 2024-03-28 233606](https://github.com/daaoooy/data-communication_project1/assets/143688136/6e8a8bc1-419f-4d1f-b895-ec408f8cc823)








# 요약

소켓을 생성하고 주소를 지정, bind 통해 UDP 통신으로 데이터를 주고 받을 수 있는 환경을 만들어 주었고 sendto() 함수와 recvfrom() 함수를 이용해 Receiver와 Sender가 데이터를 주고 받을 수 있도록 해주었다. 서로 주고 받는 과정을 보기 위해 화면에 문장을 출력해주었다. 응답을 받았다면 그 응답을 화면에, 응답을 보냈다면 보냈음을 화면에 출력해주었다.  프로그램 코드 중간 중간에 sleep(3); 을 적어주었다. sleep(3); 을 통해 주고 받는 과정을 한 단계씩 확인할 수 있었다. 파일 전송에 있어서 전송할 파일 이름과 Receiver가 받아 저장할 새로운 파일 이름은 입력을 통해 받았다. 프로그램을 실행해보면 같은 디렉토리 안에 새로운 파일이 생김을 확인할 수 있다.  두 프로그램은 Receiver가 파일 데이터를 다 받고 WellDone이라는 응답을 보내면 종료된다.








