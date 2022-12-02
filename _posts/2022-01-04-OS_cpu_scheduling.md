---
title: CPU 스케줄링
author: totoroto
date: 2022-01-04 00:00:00 +0300
categories: [OS]
tags: [OS]
render_with_liquid: false
pin: false
---

연말에 노느라 강의를 못듣고 있는데 *(핑계야...)*      
완강을 목표로 다시 달려봐야쥬~
이번 정리 포스팅은 CPU 스케줄링입니다. CPU 효율적으로 잘 굴리기...  

[KOCW 공개 강의: 반효경, 운영체제와 정보기술의 원리](http://www.kocw.net/home/cview.do?cid=4b9cd4c7178db077)   


<br>

---

# CPU 스케줄링

## 프로세스의 특성 분류

- **CPU-bound 프로세스**    
    계산 위주의 job.    
    *few very long CPU bursts.*
    
- **I/O bound 프로세스**    
    CPU를 잡고 계산하는 시간보다 I/O에 많은 시간이 필요한 job.  
    *many short CPU bursts*
    
<br>
일반적으로 프로그램 실행 시 아래와 같이 CPU burst와 I/O burst가 반복 됩니다.

```CPU burst → I/O burst → CPU burst → I/O burst → ...```

CPU를 길게 쓰는 프로그램(CPU bound job)과 CPU를 짧게 쓰는 프로그램(I/O bound job,   
I/O burst가 자주 일어나는 프로그램)이 섞여 있기 때문에 CPU 스케줄링이 필요합니다.

- I/O bound job은 주로 사람과 인터렉션하는 프로그램이므로 먼저 CPU를 줍니다.
- CPU와 I/O 장치 등 시스템 자원을 골고루 효율적으로 사용

<br>

---

## CPU 스케줄러 & 디스패처

운영체제 내에 있는 스케줄링을 하는 코드.

- **CPU 스케줄러(Scheduler)**   
    Ready 상태의 프로세스 중에서 이번에 CPU를 줄 프로세스를 고른다.
    
- **디스패처(Dispatcher)**
    CPU 스케줄러에 의해 결정된 프로세스에게 CPU를 넘긴다.   
    이 과정을 *context switch(문맥 교환)* 라고 한다.
    
- **CPU 스케줄링이 필요한 경우**    
    1. Running → Blocked (ex. I/O 요청을 하는 시스템콜)

    2. Running → Ready (ex. 타이머 인터럽트)

    3. Blocked → Ready (ex. I/O 완료 후 인터럽트)

    4. Terminate
    
    → 1, 4의 스케줄링은 강제로 빼앗지 않고 자진 반납(*nonpreemptive*)     
      나머지는 강제로 빼앗음(*preemptive*)

<br>

---

## 성능 척도(Scheduling Criteria)

CPU 스케줄링이 어떤 것이 좋은 지 판단하는 척도.

- **CPU 이용률(utilization)**   
    CPU가 놀지 않고 일한 시간의 비율
    
- **처리량(Throughput)**   
    CPU가 단위 시간 당 프로세스를 몇 개 완료 시켰는지
    
- **소요시간, 반환시간(Turnaround time)**   
    CPU를 사용한 시간 + CPU를 기다린 시간
    
- **대기 시간(Waiting time)**   
    프로세스가 ready queue에서 대기한 전체 시간
    
- **응답 시간(Response time)**  
    프로세스가 Ready queue에 들어와서 최초로 CPU를 얻기까지 걸린 시간
    
<br>

---


# 스케줄링 알고리즘(Scheduling Algorithms)

## FCFS(First-Come First-Served)
*nonpreemptive.* 먼저 도착한 것을 먼저 처리.

| 프로세스 | Burst Time |
| --- | --- |
| P1 | 24 |
| P2 | 3 |
| P3 | 3 |

- 프로세스의 도착 순서 : P1 → P2 → P3
    
    Waiting time : P1 = 0, P2 = 24, P3 = 27     
    Average waiting time : (0+24+27)/3 = 17
    
    *Convoy effect : 시간이 긴 프로세스가 먼저 도착하는 바람에 시간이 짧은 프로세스가 오래 기다리게 되는 현상*

<br>

- 프로세스의 도착 순서 : P2 → P3 → P1
    
    Waiting time : P2 = 0, P3 = 3, P1 = 6
    
    Average waiting time : (0+3+6)/3 = 3

<br>

---

## SJF(Shortest-Job-First)

CPU birst time이 가장 짧은 프로세스를 제일 먼저 스케쥴.

**minimum average waiting time**을 보장합니다.

Nonpreemptive와 Preemptive 두 가지 버전이 있습니다.

1. **Nonpreemptive**

    일단 CPU를 잡으면 이번 CPU burst가 완료될 때까지 CPU를 선점(preemption) 당하지 않음.
    
    (예시)
    
    | 프로세스 | Arrival Time | Burst Time |
    | --- | --- | --- |
    | P1 | 0.0 | 7 |
    | P2 | 2.0 | 4 |
    | P3 | 4.0 | 1 |
    | P4 | 5.0 | 4 |
    
    P1 → P3 → P2 → P4 순서로 처리가 됩니다.
    
    따라서, Average waiting time : (0+3+6+7) / 4 = 4
    
<br>

2. **Preemptive**  **더 효율적*
    
    현재 수행 중인 프로세스의 남은 birst time보다 더 짧은 CPU birst time을 가지는 새로운 프로세스가
    도착하면 CPU를 빼앗김.  
    이 방법을 SRTF(Shortest-Remaining-Time-First)라고 부른다.
    
    (예시)
    
    위와 동일한 표를 기준으로
    
    새로운 프로세스가 Ready queue에 도착할 때마다 burst time이 짧은 순서를 판단하여 실행합니다.     
    따라서, P1 → P2 → P3 → P2 → P4 → P1 순으로 처리가 됩니다.       
    Average waiting time : (9 + 1 + 0 + 2 ) / 4 = 3
    
<br>

### SJF의 단점

1. **starvation 발생**      
    짧은 프로세스에게 우선권을 부여하기 때문에 long job은 영원히 CPU를 얻지 못할 수 있습니다.
    
2. **CPU burst time을 정확히 알 수 없다.** *다음 CPU burst time의 추정만이 가능하다.*        
    → 과거, 특히 직전의 CPU burst time을 토대로 추정   

<br>

---   

## Priority Scheduling

우선순위가 높은 프로세스에게 CPU를 스케줄링. 

작은 정수(integer) 일수록 우선순위가 높다.

SJF는 일종의 priority scheduling이다. (priority == 예측된 다음 CPU burst time)

Nonpreemptive와 Preemptive 두 가지 버전이 있습니다.

- **문제점**    
    starvation : 우선순위가 낮은 프로세스는 CPU를 얻지 못할 수 있습니다.
    
- **해결책**    
    Aging: 시간이 지날수록 우선순위를 높여서 CPU를 쓸 수 있게 해줍니다.

<br>

---

## Round Robin(RR)

각 프로세스는 *동일한 할당 시간(time quantum)*을 가집니다.

할당 시간이 지나면 프로세스는 CPU를 빼앗기고(preemptive) ready queue의 제일 뒤에 다시 줄을 섭니다.

할당 시간이 길면 = FCFS     
할당 시간이 짧으면 = context switch 오버헤드가 커진다.

**(예시) time quantum = 20인 RR**   

| 프로세스 | Burst time |
| --- | --- |
| P1 | 53 |
| P2 | 17 |
| P3 | 68 |
| P4 | 24 |

P1 → P2 → P3 → P4 → P1 → P3 → P4 → P1 → P3 → P3 순으로 처리가 됩니다.

일반적으로 SJF보다 평균 turnaround time이 길지만 response time은 더 짧습니다.

*Turnaround time :  CPU를 사용한 시간 + CPU를 기다린 시간.*

*Response time : Ready queue에 들어와서 최초로 cpu를 얻는 데 걸리는 시간.*

<br>

---

## 멀티레벨 큐(Multilevel Queue)

Ready queue를 아래와 같이 여러 개로 분할

- foreground(interactive job)
- background(batch - no human interaction)

각 큐는 독립적인 스케줄링 알고리즘을 가집니다.

- foreground : RR        *// 응답 시간이 빨라야 하므로*
- background : FCFS  *// 보통 long job이 위치하므로*

<br>

큐에 대한 스케줄링이 필요..

- **Fixed priority scheduling**     
    모든 foreground가 실행 된 후, background를 실행하는 방법.
    starvation 가능성이 있습니다.   
    
- **Time slice**    
    각 큐에 CPU time을 적절한 비율로 할당하는 방법.
    ex. 80%는 foregound에 할당하고 20%는 background에 할당합니다.
    

| 우선 순위 순서(높은 순) |
| --- |
| system 프로세스 |
| interactive 프로세스 |
| interactive editing 프로세스 |
| batch 프로세스 |
| student 프로세스 |

프로세스의 우선순위가 한 번 정해지면 다른 큐로 이동할 수 없습니다.

<br>

---

## 멀티레벨 피드백 큐(Multilevel Feedback Queue)

프로세스가 다른 큐로 이동이 가능합니다.

**(예시)**

```→ 큐 (quantum = 8) → 큐(quantum = 16) → 큐(FCFS) → ..```

프로세스 A가 8ms 동안 CPU를 사용했음에도 끝나지 못했다면 다음 큐로 이동하여 16ms 동안   
사용합니다. job을 끝내지 못했다면 FCFS로 이동하면서 점점 신분이 떨어집니다..

---

## 멀티 프로세서 스케줄링(Multiple-Processor Scheduling)

CPU가 여러 개인 경우의 스케줄링.

- **Homogeneous processor인 경우**
    
    반드시 특정 CPU에서 수행되어야 하는 프로세스가 있는 경우 문제가 더 복잡해집니다.    
    Queue에 한 줄로 세워서 각 CPU가 알아서 꺼내가게 할 수 있습니다.
    
- **Load sharing**
    
    여러 CPU들이 골고루 일을 할 수 있도록 job이 특정 CPU에 몰리지 않도록 하는 메커니즘 필요.    
    별개의 큐를 두는 방법 vs 공동 큐를 사용하는 방법
    
- **Symmetric Multiprocessing(SMP)**
    
    각 프로세서가 알아서 스케줄링 결정.
    
- **Asymmetric multiprocessing**
    
    하나의 대장 CPU가 시스템 데이터의 접근과 공유를 책임지고 나머지 CPU는 거기에 따름.
    
 <br>

 ---   

## Real-time Scheduling

데드라인을 만족시켜야하는 데드라인 개념이 추가됩니다. 

- **Hard real-time systems**
    
    데드라인을 어기면 큰일 나는 task.   
    정해진 시간 안에 반드시 끝내도록 스케줄링 해야합니다.   
    이 경우, CPU 도착 시간(Arrival Time)을 미리 알고 오프라인으로 스케줄링합니다.   
    
- **Soft real-time systems**

    일반 프로세스에 비해 높은 priority를 갖도록 해야 합니다.

<br>

---

## Thread Scheduling

쓰레드 : 하나의 프로세스 안에 CPU 수행 단위가 여러 개 있는 것

- **Local Scheduling**
    
    User level thread인 경우, 프로세스 내부에서 어느 쓰레드에게 CPU를 줄 지 결정해야 합니다.
    
- **Global Scheduling**
    
    Kernel level thread인 경우, 운영체제가 쓰레드의 존재를 알고 있기 때문에 CPU 스케줄링을 할 때    
    어느 쓰레드에게 CPU를 줄 지 직접 스케줄링합니다.
    
<br>

---

## 어떤 CPU 알고리즘이 좋은 지 평가하는 방법

- Queueing models
    
    job의 도착율(arrival time)과 CPU의 처리률(service rate)이 확률 분포로 주어졌을 때 복잡한 수식을 통해 계산.
    
- 구현 & 성능 측정 (Implementation & Measurement)
    
    실제 시스템에 알고리즘을 구현하여 실제 작업에 대해서 성능을 측정 비교.
    
- **모의 실험(Simulation)**
    
    알고리즘을 모의 프로그램으로 작성하여 신빙성 있는 input을 입력하여 결과 비교.
