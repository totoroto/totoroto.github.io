---
title: 컴퓨터 시스템의 구조
author: totoroto
date: 2021-12-17 00:00:00 +0300
categories: [OS]
tags: [OS]
render_with_liquid: false
pin: false
---

[KOCW 공개 강의: 반효경, 운영체제와 정보기술의 원리](http://www.kocw.net/home/cview.do?cid=4b9cd4c7178db077)   
두 번째 차시, 정리 시작합니다.
노트에 도식 그리다 보면 아이패드 뽐뿌 오네요😂   

<br/>

---
## 컴퓨터 시스템 구조

![KakaoTalk_Photo_2021-12-14-14-31-22.jpeg](/assets/img/OS/KakaoTalk_Photo_2021-12-14-14-31-22.jpeg){: width="500" height="300"}

### Mode bit

사용자 프로그램의 잘못된 수행으로 다른 프로그램 및 운영체제에 피해가 가지 않도록 하기 위한 보호장치  
- Mode bit을 통해 하드웨어적으로 두 가지 모드의 operation 지원
   
    - 1 [사용자 모드] : 사용자 프로그램 수행
    - 0 [모니터 모드] : OS 코드 수행
        - 보안을 해칠 수 있는 중요한 명령어는 모니터 모드에서만 수행 가능(특권 명령)
        - 인터럽트나 익셉션 발생 시 하드웨어가 mode bit을 0으로 바꿈
        - 사용자 프로그램에게 CPU를 넘기기 전에 mode bit을 1로 세팅

*Note*

- 디스크 파일을 다 읽으면 디스크 컨트롤러가 CPU에게 인터럽트를 겁니다.  
    CPU는 기계어 수행이 끝나면 인터럽트가 들어왔나 체크를 합니다.   
    프로그램 카운터는 운영체제를 가리킵니다. 그러면 mode bit이 0됩니다.
    
<br/>

### 프로그램 카운터

- CPU에 붙어있는 레지스터. 다음 번 실행할 기계어의 메모리 주소를 가지고 있습니다.   
    ex. 프로그램 카운터가 사용자 프로그램 A를 가리키고 있으면 → 프로그램 A가 CPU에서 실행 중
    
<br/>

### 타이머(Timer)

- 정해진 시간이 흐른 뒤 운영체제에게 제어권이 넘어가도록 인터럽트를 발생 시킴.
   
- 일정 시간이 흐르면 타이머가 CPU에게 인터럽트를 걸기 때문에 만약 프로그램 A가 무한루프가 돌면서
CPU를 독점하더라도 막을 수 있습니다.

<br/>

### 시스템콜(System call)

- 사용자 프로그램이 OS의 서비스를 받기 위해 커널 함수를 호출하는 것.
    - 사용자 프로그램이 CPU를 가지고 있는 상태에서는 특권 명령(ex. I/O요청 : 디스크에서 파일을 읽어와달라)을 사용하지 못하기 때문에     
    스스로 운영체제를 불러서 대신 요청.   
    
- 소프트웨어 인터럽트 : 프로그램이 자신의 코드를 통해 인터럽트를 건다

<br/>

### 인터럽트

- 인터럽트를 당한 시점의 레지스터와 프로그램 카운터를 저장한 후 CPU의 제어를 인터럽트 처리 루틴에 넘긴다.
    - 인터럽트(하드웨어 인터럽트)
    - Trap(소프트웨어 인터럽트) → Exception(프로그램 오류), System call(프로그램이 커널함수 호출)
       
- 인터럽트 관련 용어    
    인터럽트 벡터 : 해당 인터럽트의 처리 루틴 주소를 가지고 있음    
    인터럽트 처리 루틴(인터럽트 핸들러) : 해당 인터럽트를 처리하는 커널 함수    
       
- 운영체제는 인터럽트가 들어와야 구동됩니다.

<br/>

### 디바이스 컨트롤러

- 해당 I/O 장치 유형을 관리하는 일종의 작은 CPU
   
- I/O작업이 끝났으면 디바이스 컨트롤러가 CPU에게 인터럽트를 걸어 작업이 끝났음을 알려줍니다.

<br/>

### 동기식 입출력과 비동기식 입출력

- 동기식 입출력     
    [방법 1]    
        CPU가 I/O요청을 하고나서 I/O가 작업 완료 후 인터럽트를 걸었을 때 그 결과를 보고 그 다음 스텝을 밟음.    
        but, I/O가 끝날 때 까지 CPU를 낭비 시킴     

    [방법 2]    
        1. I/O가 완료 될때까지 해당 프로그램에게서 CPU를 빼앗음      
        2. I/O 처리를 기다리는 줄에 그 프로그램을 줄 세우고 다른 프로그램에게 CPU를 줌
- 비동기식 입출력   
    CPU가 I/O요청을 하고난 후 디스크가 뭘 하던지 상관없이 그냥 다음 일을 함.

<br/>

### DMA(Direct Memory Access)

- DMA 컨트롤러 : 직접 메모리에 접근할 수 있는 컨트롤러.
    - CPU에 인터럽트가 너무 자주 걸리는 것을 막기 위한 용도.    
    고속 I/O장치의 경우, 인터럽트가 자주 걸립니다.

    - I/O 작업이 바이트(byte) 단위로 이루어지는 것이 아니고 block단위로 DMA가
        메모리에 복사하는 작업까지 다 해주고 나서   
        한 번에 알려주는 인터럽트를 발생 시킵니다.

<br/>

---
## 프로그램의 실행 (메모리 load)

![KakaoTalk_Photo_2021-12-14-17-05-18.jpeg](/assets/img/OS/KakaoTalk_Photo_2021-12-14-17-05-18.jpeg){: width="500" height="500"}

<br/>

- **주소 변환(Address translation)**    
 가상 메모리의 논리 주소를 물리적 메모리의 물리 주소로 변환   

<br/>
 
- **가상 메모리(Virtual memory)**   
    - code : CPU에서 수행할 기계어가 위치. ex. 메인함수, 사용자 정의 함수 —> 컴파일러 —> 기계어
    - data : 데이터가 보관되는 영역. ex. 전역 변수, 프로그램이 시작되서 끝날 때 까지 존재하는 값
    - stack : 함수 호출과 리턴되는 정보를 스택에 쌓아둡니다.

<br/>

- **커널 주소공간의 내용**  
    - code
        - 시스템 콜, 인터럽트 처리 코드
        - 자원 관리를 위한 코드
        - 편리한 서비스 제공을 위한 코드
    - data
        - 모든 하드웨어(CPU, mem, disk)를 관리하기 위한 자료구조를 가지고 있음.
        - 현재 실행 중인 모든 프로세스를 관리하기 위한 자료구조(PCB)를 가지고 있음.     
          예를 들어, 프로세스가 3개 → PCB가 3개
    - stack
        - 각 프로세스마다 별도의 커널 스택을 가지고 있습니다.
        - 프로그램 A를 사용하다가 커널로 넘어왔으면 '프로세스 A의 커널 스택'을 사용합니다.

<br/>

## 사용자 프로그램 A가 사용하는 함수

- 사용자 정의 함수 : 자신의 프로그램에서 정의한 함수.
- 라이브러리 함수 : 자신의 프로그램에서 정의하지 않고 갖다 쓴 함수.     
    → 사용자 정의 함수나 라이브러리 함수는 프로세스 A의 주소 공간에 저장된다.
    
- 커널 함수 : 운영체제 프로그램의 함수, 커널 함수의 호출을 시스템 콜이라 한다.      
    → CPU 제어권을 운영체제에게 넘긴 후(커널 모드) 커널의 주소공간에 저장된다.