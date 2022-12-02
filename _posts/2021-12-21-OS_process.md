---
title: 프로세스의 개념과 상태
author: totoroto
date: 2021-12-21 00:00:00 +0300
categories: [OS]
tags: [OS]
render_with_liquid: false
pin: false
---

나름 꾸준히 듣는 중...  
3장 프로세스 시작.   
[KOCW 공개 강의: 반효경, 운영체제와 정보기술의 원리](http://www.kocw.net/home/cview.do?cid=4b9cd4c7178db077)   

<br/>

---

## 프로세스의 개념

- 프로세스 : 실행 중인 프로그램.

- 프로세스의 문맥(context) : 프로세스가 현재 상태를 나타내는 정보

    - CPU 수행 상태를 나타내는 하드웨어 문맥  // *CPU에서 어디까지 수행했는가*
        - 프로그램 카운터, 각종 레지스터
    - 프로세스의 주소 공간
        - code, data, stack
    - 프로세스 관련 커널 자료 구조
        - PCB(Process Control Block), Kernel stack

<br/>

## 프로세스의 상태

![KakaoTalk_Photo_2021-12-15-11-41-04.jpeg](/assets/img/OS/KakaoTalk_Photo_2021-12-15-11-41-04.jpeg){: width="600" height="300"}

<br/>

- 프로세스는 상태(state)가 변경되며 수행됩니다.   
    - **Running**   
        CPU를 잡고 instruction을 수행 중인 상태     
    - **Ready**     
        CPU를 기다리는 상태(메모리 등 다른 조건을 모두 만족 하고)

    - **Blocked(wait, sleep)**      
        CPU를 주어도 당장 instruction을 수행할 수 없는 상태     
        프로세스 자신이 요청한 이벤트(ex. I/O)가 즉시 만족되지 않아 이를 기다리는 상태   
            ex. 디스크에서 file을 읽어와야 하는 경우  

    - **New** : 프로세스가 생성 중인 상태   

    - **Terminated** : 수행(execution)이 끝난 상태  

<br/>

### 프로세스 상태도

![https://z-images.s3.amazonaws.com/thumb/d/d5/Process-state.jpg/600px-Process-state.jpg](https://z-images.s3.amazonaws.com/thumb/d/d5/Process-state.jpg/600px-Process-state.jpg)

*interrupt : 타이머 인터럽트*

<br/>

---

## PCB(Process Control Block)

- 운영체제가 각 프로세스를 관리하기 위해 프로세스 당 유지하는 정보

- 커널의 주소공간에 저장됨
- 구성 요소
    1. OS가 관리 상 사용하는 정보
        - Process state, Process ID
        - scheduling information, priority
    2. CPU 수행 관련 하드웨어 값
        - Program counter, registers
    3. 메모리 관련
        - Code, Data, Stack의 위치 정보
    4. 파일 관련
        - Open file descriptors *// 프로세스가 오픈해서 쓰고있는 파일이 어떤 것이 있는지*

<br/>

### 문맥 교환(Context Switch)

- CPU를 한 프로세스에서 다른 프로세스로 넘겨주는 과정.

- CPU가 다른 프로세스에게 넘어갈 때 운영체제는 다음을 수행합니다.   
    - CPU를 내어주는 프로세스의 상태를 그 프로세스의 PCB에 저장
        → 나중에 CPU를 얻었을 때 그 다음 위치부터 실행하기 위해서
        
    - CPU를 새롭게 얻는 프로세스의 상태를 PCB에서 읽어옴

- System call이나 Interrupt 발생 시 반드시 context switch가 일어나는 것은 아닙니다.
    - 예시

        1. 사용자 프로세스A → 시스템콜 or ISR 함수 → 사용자 프로세스A
            - user mode → kernel mode → user mode *(문맥 교환 없이 user mode 복귀)*

        2. 사용자 프로세스A → 타이머 인터럽트 or I/O요청 시스템콜 → 사용자 프로세스 B
            - user mode → kernel mode → user mode *(문맥 교환 일어남)*
        
        → 1 의 경우에도 CPU 수행 정보 등 context의 일부를 PCB에 save를 해야하지만   
        문맥 교환을 하는 2 의 경우 캐시 메모리를 지워야 하므로 오버헤드가 훨씬 큽니다.
        
<br/>

### 프로세스를 스케줄링 하기 위한 큐

- Job queue : 현재 시스템 내에 있는 모든 프로세스의 집합

- Ready queue : 현재 메모리 내에 있으면서 CPU를 잡아서 실행되기를 기다리는 프로세스의 집합

- Device queue : I/O device의 처리를 기다리는 프로세스의 집합

    → 프로세스들은 각 큐를 오가며 수행됩니다.

(참고) 각 큐의 header에 PCB들이 링크드 리스트의 형태로 연결되어 있습니다.

<br/>

---

## 스케줄러

운영체제에서 스케줄링을 하는 코드

#### 장기 스케줄러(메모리 스케줄러)    // *옛날 꺼. 요즘엔 swapper를 씁니다*
- 프로세스에 메모리(및 각종 자원)을 주는 문제
    - 시작 상태의 프로세스 중 어떤 것들을 ready queue로 보낼 지 결정합니다.     
        -> 프로세스 상태도 상에서 admitted를 시켜주는 것이 장기 스케줄러

    - 시분할(time-sharing) 시스템에는 보통 장기 스케줄러가 없음(무조건 ready)

    - 메모리에 프로그램을 몇 개 올릴 지 제어
   
<br/>

#### 단기 스케줄러(CPU 스케줄러)
- 어떤 프로세스에게 CPU를 줘서 다음번에 running 시킬 지 결정합니다.
- 타이머 인터럽트 걸릴때마다 자주 호출이 됩니다. (ms 단위)

<br/>

#### 중기 스케줄러(Swapper)
- 프로세스에게서 메모리를 뺏는 문제
- 프로세스에게 일단 메모리를 주고 메모리가 부족한 경우, 통째로 프로세스를 디스크로 쫓아냅니다.

<br/>

---

## 프로세스의 상태

위 스케줄러의 개념이 도입되며 기존 Running, Ready, Blocked의 상태에 Suspended가 추가됩니다.

- Running   
    CPU를 잡고 instruction을 수행 중인 상태     

- Ready     
    CPU를 기다리는 상태(메모리 등 다른 조건을 모두 만족 하고)   

- Blocked(wait, sleep)  
    I/O 등의 이벤트를 기다리는 상태.    
    프로세스 자신이 요청한 이벤트(ex. I/O)가 즉시 만족되지 않아 이를 기다리는 상태  
        ex. 디스크에서 file을 읽어와야 하는 경우
        
- **Suspended(stopped)**    
    외부적인 이유로 프로세스의 수행이 정지된 상태   
    프로세스는 통째로 디스크에 swap out 됩니다.
        
    예) 사용자가 프로그램을 일시 정지 시킨 경우, 시스템이 여러 이유로 프로세스를 잠시 중단 시킴
        
<br/>

### Blocked vs Suspended

둘 다 CPU를 얻지 못한다는 공통점이 있습니다.

- Blocked : 자신이 요청한 이벤트가 만족되면 Ready. (프로세스가 실행 중)
- Suspended : 외부에서 resume을 해주어야 Active됩니다. (프로세스가 정지 된 상태)

<br/>

---

### 프로세스 상태도
![KakaoTalk_Photo_2021-12-15-15-09-00.jpeg](/assets/img/OS/KakaoTalk_Photo_2021-12-15-15-09-00.jpeg){: width="700" height="500"}

*Note*

- swap in : 메모리에 올라감 / swap out : 메모리에서 쫓겨남.

- I/O 작업(Blocked)을 하다가 Suspended 상태가 되면 I/O 작업이 끝났을 때 Suspended Ready 상태가 됩니다.

- 프로그램A가 실행 중인데 디스크 컨트롤러가 프로그램B의 I/O 요청이 완료되어 인터럽트가 들어왔습니다.        
    이 때, A의 상태는 여전히 커널모드에서 Running한다고 간주합니다.

<br/>

---
## Thread
  ![KakaoTalk_Photo_2021-12-15-16-07-39.jpeg](/assets/img/OS/KakaoTalk_Photo_2021-12-15-16-07-39.jpeg){: width="350" height="400"}
#### 쓰레드 : 프로세스 내부에서 CPU 수행 단위
  
- 프로세스 → [stack, code, data]로 나뉘어져 있는 주소공간을 갖고 있습니다.  

- 구현 방식에 따라 조금 다를 수 있지만..
웹 브라우저를 n개 띄우는 경우, n개의 프로세스를 만드는 것은 비효율적입니다.     
따라서 쓰레드를 사용해 웹 브라우저를 여러 개 띄워도 code 부분은 share하고 1개 프로세스가 만들어집니다.
            
    → 웹 브라우저가 무슨 일을 하는 가에 따라 PCB내 레지스터 값(thread 1, thread 2, thread 3..)과    
    주소 공간 내 stack(thread 1의 스택, thread 2의 스택, thread 3의 스택..) 다른 일을 하게 함.
                
    → 프로세스 간 이동을 하는 Context Switch에 비해 쓰레드 간 이동은 오버헤드가 적습니다. *효율적*
            
<br/>

#### 쓰레드의 구성
 - 프로그램 카운터
 - register set
 - stack space

<br/>

#### 쓰레드가 동료 쓰레드와 공유하는 부분 (= task)
- code section
- data section
- OS 리소스
-> 다중 스레드에서는 하나가 blocked 상태인 동안에도 다른 스레드가 running되어 빠른 처리를 할 수 있습니다.

<br/>

#### 쓰레드의 장점
- 응답이 빠름. 자원의 공유. 프로세스 여러 개 보다 쓰레드 여러 개가 오버헤드가 적습니다.
- 멀티 프로세서 환경에서 스레드 여러 개를 두면 더 빨리 작업할 수 있습니다.
