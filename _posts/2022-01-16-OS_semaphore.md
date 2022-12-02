---
title: Semaphore
author: totoroto
date: 2022-01-16 00:00:00 +0300
categories: [OS]
tags: [OS]
render_with_liquid: false
pin: false
---

글 저장 공간 만든 후 OS 포스팅만 주구장창했네요..  
다른 주제도 다루고 싶은데 이렇게 OS는 또랑으로 빠지는건가🤭  
남은 챕터가 많지 않으니 화이팅~  
오늘은 여러 프로세스들이 공유 자원에 접근할 때 동기화 문제를 해결하기 위해 쓰는 세마포어에 대해 끄적여 보겠습니다.  

[KOCW 공개 강의: 반효경, 운영체제와 정보기술의 원리](http://www.kocw.net/home/cview.do?cid=4b9cd4c7178db077)   

---

## 세마포어(Semaphores)
일종의 추상 자료형. 

**(예시) 세마포어 변수 S**

- S는 integer 값

- 자원을 획득하는 과정(lock)인 P연산과 반납하는 과정인 V연산이 있다. *atomic*

```c
P(S) example
// S = 5, 자원이 5개. 
// 프로세스 5개가 각각 자원을 쓴다면 특정 프로세스가 특정 자원을 바로 못쓰고 기다려야 합니다.
while(S <= 0) wait;   
S--;
```
```c
V(S) example
S++;
```

<br>

### Busy-wait 방식의 구현

```c
// mutex == mutual exclusion 
semaphore mutex = 1; // 1개가 크리티컬 섹션에 들어갈 수 있다

do {
	P(mutex); // mutex가 양수이면 감소시킨 후 진입한다. 그렇지 않다면 wait
	/* critical section */
	V(mutex); // 세마포어 증가시킨다
	...
} while(1);
```

P 연산에서의 busy-wait(spin lock) 문제가 있습니다. -> 해결 : Block / Wake up

<br>

### Block / Wake up 방식의 구현

세마포어를 일종의 구조체 처럼 정의.  
sleep lock 방식으로 구현, lock이 걸렸을 때 슬립 시켜버린다.

```c
typedef struct {
	int value;         // semaphore
	struct process* L; // 프로세스 wait queue
} semaphore;
```

- **block**
    
    커널은 block을 호출한 프로세스를 suspend 시킨다.  
    이 프로세스의 PCB를 세마포어에 대한 wait queue에 넣음.
    
- **wakeup(P)**
    
    block된 프로세스 P를 wake up.  
	이 프로세스의 PCB를 ready queue로 옮김.
    

```c
P(S) example

S.value--;
if S.value < 0 { // 이미 누군가 자원을 쓰고 있는 상태
	/* S.L에 프로세스를 추가 합니다. */
	block();
}
```
```c
V(S) example

S.value++;
if S.value <= 0 { 
	/* S.L에서 프로세스 P를 제거 합니다. */
	wakeup(P) // block -> ready
}
```

<br>

---

### 세마포어의 두 가지 타입

- **Counting Semaphore**
    
    도메인이 0 이상인 임의의 정수 값.  
    주로 자원(resource) counting 에 사용.
    
- **Binary Semaphore (= mutex)**
    
    0 또는 1값만 가질 수 있는 semaphore  
    주로 mutual exclusion(lock/unlock)에 사용.
    

---

## 데드락과 Starvation

- **데드락(Deadlock)**  
    둘 이상의 프로세스가 상대방에 의해 충족될 수 있는 이벤트를 무한히 기다리는 현상.
    

    *(예시)세마포어 S, Q를 둘다 얻어야 일을 할 수 있는 프로세스*
    
    P0 : P(S) → P(Q) → ... → V(S) → V(Q)  
    P1 : P(Q) → P(S) → ... → V(Q) → V(S)
    
- **Starvation**
    
    *Indefinite blocking.*  
    프로세스가 suspend된 이유에 해당하는 세마포어 큐에서 빠져나갈 수 없는 현상.
    
<br>

---

## 동기화와 관련된 전통적인 문제들

### 생산자-소비자 문제 (Bounded-Buffer Problem)

![KakaoTalk_Photo_2022-01-16-21-28-02.jpeg](/assets/img/OS/KakaoTalk_Photo_2022-01-16-21-28-02.jpeg){: width="200" height="150"}

- Bounded-Buffer : 크기가 유한한 버퍼 ***공유 버퍼*

- **생산자 프로세스** : 데이터를 만들어서 버퍼에 넣는 역할

    1. Empty 버퍼가 있나요? (없으면 기다림)
    2. 공유 데이터에 lock을 건다
    3. Empty 버퍼에 데이터 입력 및 버퍼 조작
    4. Lock을 푼다
    5. Full buffer 하나 증가
    
<br>

- **소비자 프로세스** : 데이터를 꺼내는 역할

    1. Full 버퍼가 있나요? (없으면 기다림)
    2. 공유 데이터에 lock을 건다
    3. Full 버퍼에서 데이터를 꺼내고 버퍼 조작
    4. Lock을 푼다
    5. empty buffer 하나 증가

생산자 프로세스가 버퍼가 비어있는 것에 데이터를 넣으려 시도 중에 CPU를 빼앗기고 그 자리에  
또 다른 생산자 프로세스가 접근하는 공유 데이터 문제가 발생할 수 있으므로 lock을 겁니다. **소비자 프로세스도 동일

<br>

[문제점]

생산자 → 빈 버퍼가 없으면 빈 버퍼가 생길 때 까지 기다려야 합니다.  
소비자 → 내용이 들어있는 버퍼가 없으면 자원이 생길 때 까지 기다려야 합니다.

- Synchronization variables
    - **mutual exclusion** → binary 세마포어 필요!  
        shared data의 mutual exclusion을 위해   *lock/unlock*
        
    - **resource count** → counting 세마포어 필요.  
        남은 full/empty 버퍼 표시하는 용도
        

```c
semaphore full = 0
semaphore empty = n
semaphore mutex = 1

/* Producer example*/
do {
	// 아이템 x를 produce 합니다
	P(empty)
	P(mutex)
	..
	// 버퍼에 x를 추가합니다.
	..
	V(mutex)
	V(full)
} while(1)

/* Consumer example */
do {
	P(full)
	P(mutex)
	// 버퍼에서 아이템 y를 삭제합니다.
	..
	V(mutex)
	V(empty)
	..
	// 아이템 y를 consume합니다.
} while(1)
```

<br>

---

### Readers-Writers 문제

lock을 걸면 되지만 이것보다 효율성을 높여보자..!

공유데이터 DB에 대해 어떤 프로세스가 write 중에는 접근을 하지 못하도록 합니다.  
read는 동시에 여럿 해도 됩니다.

```
1. Writer가 DB에 접근 허가를 아직 얻지 못한 상태 → 모든 대기중인 Reader들은 모두 DB에 접근 가능

2. Writer는 대기 중인 Reader가 하나도 없을 때 DB접근이 허용된다.

3. Writer가 DB에 접근 중이면 Reader들은 접근이 금지된다.

4. Writer가 DB에서 빠져나가야만 Reader의 접근이 허용된다.
```

- **Shared Data**
    - DB 자체
    - readcount  // 현재 DB에 접근적인 reader의 수

<br>
- **Synchronization variables**
    - mutex       // 공유 변수 readcount를 접근하는 코드(critical section)의 mutex를 보장하기 위해 사용
    - db             // reader와 writer가 공유 db 자체를 올바르게 접근하게 하는 역할

```c
/* Writer */
P(db)
// db를 writing을 합니다.
V(db)
```
```c
/* Reader */         /* 동시에 접근 가능 */
P(mutex)
readcount++

if(readcount == 1) { P(db) } // readcount가 1보다 큰경우 lock을 따로 걸 필요 없이 다음 진행
V(mutex)
..
// db를 reading을 합니다.
..
P(mutex)
readcount--

if(readcount == 0) { V(db) }
V(mutex)
```

[문제점]

starvation 발생 가능 → reader가 끊임없이 도착하는 경우, writer는 reader가 다 읽을때까지 기다려야 합니다.

해결 : 일정 시간 내에 도착한 reader들에게만 동시에 접근을 할 수 있도록 허용.

<br>

---

### Dining Philosophers(식사하는 철학자) 문제

- 철학자
    
    think(), eat()을 수행. eat을 수행하는 주기가 각각 다릅니다.  
    밥을 먹기 위해서는 젓가락 한 쌍이 필요한데 젓가락이 공유 데이터입니다.
    

```c
semaphore chopstick[5]  // 1로 초기화

do {
	P(chopstick[i])
	P(chopstick[(i+1) % 5])
	..
	eat()
	..
	V(chopstick[i])
	V(chopstick[(i+1) % 5])
	..
	think()
	..
} while(1)
```

[문제점]

철학자가 starvation되는 문제가 발생하면 안됩니다.  
*모든 철학자가 왼쪽의 젓가락을 잡는 경우 - deadlock*

<br>

[해결 방안]

1. 4명의 철학자만 테이블에 앉히자.

1. 젓가락을 두 개 모두 집을 수 있을 때만 젓가락을 집을 수 있게 한다.   *→ 다음 챕터인 Monitor 개념을 통해 구현해보도록 하자.*
    
    **(예시) 세마포어 개념을 통해 해결한 경우, Philosopher i의 입장**
    
    ```c
    enum { thinking, hungry, eating } state[5];
    semaphore self[5] = 0 // 0 : 사용할 수 없는 상태
    semaphore mutex = 1
    
    do {
    	pickup(i)
    	eat()
    	putdown(i)
    	think()
    } while(1)
    
    void pickup(int i) {
    	P(mutex)
    	state[i] = hungry
    	test(i)
    	V(mutex)
    	P(self[i])      // point!
    }
    
    void test(int i) {
    	if (state[(i+4)%5] != eating && state[i] == hungry 
	                                    && state[(i+1)%5] != eating) {
    		state[i] = eating
    		V(self[i])  // point!
    	}
    }
    
    void putdown(int i) {
    	P(mutex)
    	state[i] = thinking
    	test((i+4) %5)
    	test((i+1) %5)
    	V(mutex)
    }
    ```
    
2. 짝수 철학자는 왼쪽 젓가락부터 잡도록하고 홀수 철학자는 오른쪽 젓가락 부터 잡도록 한다.
    
    **(예시) 앞선 세마포어 예시와 동일한 개념**
    
    P0 : P(S) → P(Q) → ... → V(S) → V(Q)
    
    P1 : P(Q) → P(S) → ... → V(Q) → V(S)