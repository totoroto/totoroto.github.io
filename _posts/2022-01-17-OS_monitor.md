---
title: Monitor
author: totoroto
date: 2022-01-17 00:00:00 +0300
categories: [OS]
tags: [OS]
render_with_liquid: false
pin: false
---

세마포어 내용이 너무 길어지는 것 같아 분리해서 올려요~ 또 다른 동기화 수단으로 Monitor에 대해 다루어 봅니다.

[KOCW 공개 강의: 반효경, 운영체제와 정보기술의 원리](http://www.kocw.net/home/cview.do?cid=4b9cd4c7178db077)   

---
## Semaphore의 문제점

- 코딩하기 힘들다

- 정확성(correctness)의 입증이 어렵다

- 자발적 협력(voluntary cooperation)이 필요하다

- 한 번의 실수가 모든 시스템에 치명적 영향


**(예시) Mutual exclusion이 깨짐**

```c
V(mutex)
/* Critical Section */
P(mutex)
```

**(예시) Deadlock**

```c
P(mutex)
/* Critical Section */ 
P(mutex)
```
<br>

---

## Monitor

- 고급 언어 차원에서 제공하는 동기화 수단.

- 공유 데이터를 접근할 땐 모니터 내에 프로시저(procedure)를 통해서만 접근할 수 있다.

- 공유 데이터에 대한 동시 접근을 Monitor가 책임짐.  

	→ 세마포어는 연산을 제공할 뿐, 공유데이터에 대한 접근은(lock을 걸고 품) 프로그래머가 책임짐.
    

```c
monitor monitor-name {
	shared variable
	
	procedure body P1(...) { }
	procedure body P2(...) { }
	procedure body P3(...) { }

	init {}
}
```

- 프로그래머가 동기화 제약 조건을 명시적으로 코딩할 필요가 없다

- Monitor 내에서는 한 번에 하나의 프로세스만 활동 가능하다.  
→ 또 다른 프로세스들은 queue에 줄 세워서 진입을 막음으로써 동기화 문제 해결.
    
- 자원의 여분이 없어서 프로세스가 잠들어서 큐에서 기다릴 땐 condition variable을 사용
    
    **wait()** : 다른 프로세스가 signal()을 호출하기 전 까지 suspend 됩니다.
    
    **signal()**: 하나의 suspend된 프로세스를 resume 합니다. suspend된 프로세스가 없으면 아무 일도 일어나지 않습니다.

<br>    

## Bounded-buffer 문제

*세마포어 코드와 비교해서 보세요~*

```c
monitor monitor_name {
	int buffer[N]; // 공유 버퍼
// condition var은 값을 가지지 않고 자신의 큐에 프로세스를 매달아 sleep또는 깨우는 역할만 함
	condition full, empty; 

	void produce(int x) {
		if /* 빈 버퍼 가 없을 때 */  {
			// 빈 버퍼가 생길 때 까지 condition var 내에서 프로세스는 blocked
			empty.wait(); 
		}
		/* 버퍼에 x 추가 하는 코드 */

		full.signal();
	}

	void consume(int* x) {
		if /* 풀 버퍼가 없을 때 */ {
			full.wait();
		}
		/* 버퍼 내에서 아이템 삭제 및 그것을 x에 저장하는 코드 */

		empty.signal();
	}
}
```

<br>

---

## 식사하는 철학자 문제

철학자 i는 젓가락을 잡고 양옆의 철학자가 밥을 먹지 않는 상태라면 **signal()** 수행, 

양옆에 철학자가 밥을 먹고 있다면 **wait()**를 수행합니다.

결국, **signal()**을 수행하게 되면 밥을 먹게(eat)되고 그 후 젓가락을 내려 놓습니다(put down).

젓가락을 내려놓으면서 인접한 양 옆의 철학자가 밥을 먹을 수 있는지 확인(test)합니다.

<br>

그리고 위 과정을 동일한 방식대로 수행하며 queueing된 각각의 프로세스를 차례로 깨우며 수행하는 방식입니다.

글로 쓰니 복잡한데 아래 sudo코드를 보면 이해하기 쉬워요.

```c
Each Pilosopher: { // 철학자들은 먹거나 생각하거나...
	pickup(i) // 젓가락을 잡는다 = 공유데이터 접근
	eat()
	putdown() // 젓가락을 내려놓는다
	think()
} while(1)


/* ----------------------------------- */


monitor dining_philosopher {
	enum { thinking, hungry, eating } state[5];

	condition self[5]; // condition variable

	void pickup(int i) {
		state[i] = hungry;
		test(i);
		if state[i] != eating {       // 밥을 먹지 못한 상태: test 메소드에서 조건문 만족X
			self[i].wait();       // 프로세스 i는 기다린다..
		}
	}


	void test(int i) {
	// 왼쪽 오른쪽 철학자가 밥을 먹지 않다면
		if( (state(i+4)%5 != eating && state[i] == hungry  
		&& state[(i+1)%5] != eating)) {
			state[i] = eating
			self[i].signal(); // 프로세스 i를 깨운다!
		}
	}


	void putdown(int i) {
		state[i] = thinking;
		
		// 인접한 양옆의 철학자에 대해 밥 먹을 수 있는 지test
		test((i+4) %5);
		test((i+1) %5);
	}


	void init() {
		for(int i=0; i<5; i++) {
			state[i] = thinking;
		}
	}
}
```