# [OS] 디스크 스케줄링 & RAID

## 1. 디스크의 구조

SSD, CD, DVD 등의 저장장치는 다음과 같은 구조를 띤다.

**플래터(원판)**이 여러개의 **트랙(실린더)**으로 이루어지고, 각 트랙은 여러개의 **섹터**로 구성된다.

고정축은 고정되어 있고, 각각의 고정축에 존재하는 **헤드**가 축 위에서 앞뒤로 움직이며 섹터의 데이터를 읽는 방식이다.

헤드가 움직이는 것 같지만, 실제로는 **암**이 움직여 헤드를 원하는 위치로 이동시킨다.

<img width="732" alt="image" src="https://github.com/user-attachments/assets/24efa948-ccc0-4f9a-97e9-2ab833163bcf">


## 2. 디스크 탐색 퍼포먼스

디스크를 탐색하는 시간은 세 가지가 결정한다.

- 탐색시간(Seek time) : 현 위치에서 특정 실린더(트랙)로 **디스크 헤드가 이동**하는 데 소용되는 시간
- 회전 지연시간(Rotation delay time) : 가고자 하는 **섹터가 디스크 헤드까지 도달**하는 데 걸리는 시간
- 전송시간(Transfer time) : 데이터를 전송하는 데 걸리는 시간

<img width="734" alt="image" src="https://github.com/user-attachments/assets/6aad12b5-f1b1-4baf-8fe7-041c21d8c651">


## 3. 디스크 스케줄링

그리고 디스크 탐색 과정에서, **탐색 시간(Seek time)**이 제일 오래 걸리는 과정이므로, 이 과정을 줄이기 위해 진행하는 것이 **디스크 스케줄링(Disk Scheduling)**이다.

→ 헤드가 트랙 사이를 이동할 때, 최소한의 이동거리를 보장하는 과정!

> 아래의 예시들은, 트랙의 크기가 200인 디스크를 기준으로 한 예시이다.
현재 디스크 헤드의 위치는 100이며, 디스크 스케줄러가 요청받은 트랙은 다음과 같은 순서로 큐에서 대기중이다.
[55, 58, 39, 18, 90, 160, 150, 38, 184]
> 

### 3.1. FCFS, FIFO 스케줄링

<img width="724" alt="image" src="https://github.com/user-attachments/assets/fc4bc894-55dc-4205-8090-48337051ff21">


- 가장 먼저 도착한 요청(트랙)을 우선적으로 처리하는 방식
- 장점 : starvation이 발생하지 않음.
- 단점 : 성능이 좋지 않음.

### 3.2. SSTF(Shortest Service Time First)

<img width="726" alt="image" src="https://github.com/user-attachments/assets/66b056c5-ae45-46fc-ad93-7452d508b2a1">


- 현재 헤드에서, 가장 가까운 요청(트랙)을 우선적으로 처리하는 방식
- 장점 : 성능이 제일 좋다
- 단점 : starvation이 발생할 수 있다.

### 3.3. SCAN

<img width="739" alt="image" src="https://github.com/user-attachments/assets/ea3b9796-28b4-47db-9052-d397c41b0fef">


- 암이 한 방향으로만 이동하며, 요청(트랙)을 읽어들이는 방식이다.
- 장점 : starvation이 거의 발생하지 않는다.
    - 한 트랙에서 지속적 요청이 들어올 경우 발생
- 단점 : SSTF보다 성능이 떨어진다.
          디스크의 가운데에 위치한 트랙에서 처리를 더 자주 받게 되므로, **공평하지 않다.**

### 3.4. C-SCAN ( Circular-SCAN)

<img width="732" alt="image" src="https://github.com/user-attachments/assets/d2fa2d64-a262-40a7-a1e9-fa83deaad0c2">


- SCAN방식처럼 한 방향으로 이동하되, 디스크의 끝에 도달 시 **반대편 끝으로 이동하여(이동 중에는 해당 트랙의 요청을 처리하지 않음)** 다시 처리하는 방식이다.
- 장점 : starvation이 거의 발생하지 않는다.
          완벽히 fair한 방식이다
- 단점 : 암이 이동하는 시간동안 처리를 하지 않으므로, SCAN보다 성능이 떨어진다.

### 3.5.  N-Step SCAN

[55, 58, 39, 18, 90, 160, 150, 38, 184]
→ [55, 58, 39], [18, 90, 160], [150, 38, 184]

- 요청을 크기가 N인 **서브큐**로 나눈 뒤 각 큐에 대해 SCAN방식으로 처리하는 방식
→ **N 값을 잘 정하는 것이 중요하다.**
- 장점 : starvation이 없다.
- 단점 : N의 값에 따라, 서브큐가 많아지는 문제 또는 공정성 문제가 발생 가능하다.

### 3.6. F-SCAN

Queue 1 : [55, 58, 39, 18, 90, 160, 150, 38, 184] (처리중…)
Queue 2: []

- 2개의 큐를 만들고, 하나의 큐의 요청을 처리하는 동안 들어온 요청은 다른 큐에 넣는다.
- 각 큐에서는 SCAN방식을 사용한다.
- starvation이 발생하지 않고, SCAN방식과 거의 비슷하다.

### 3.7. Priority Scheduling

- 헤드의 이동거리를 고려하지 않고, 프로세스의 종류에 따라 우선순위를 관리함\
- 실행시간이 짧거나, 협업에 필요한 프로세스를 먼저 실행함.

### 3.8. Last-in-First-out

- 최근에 들어온 요청부터 먼저 처리한다.
- 왜? Locality를 이용하기 위해!
- Starvation 발생 가능성 UpUpUpUp

## 4. RAID란?

RAID : Redundant Array of Independent Disks

RAID는 여러개의 디스크를 사용하여, 저장 장치의 **성능**과 **신뢰성**을 향상시키는 기술이다.

- 성능 : N개의 요청이 들어왔을 때, 하나의 디스크에서 처리하는 것이 아니라 여러개의 디스크에서 요청을 분산해서 가져가므로 더 빠른 처리가 가능하다.
- 신뢰성 : RAID의 다양한 수준을 통해 복구가 가능하도록 할 수 있다.
- 즉, RAID를 통해서 디스크의 **기계적인 장애**로부터 사용자의 데이터를 **안정적**으로 지킬 수 있다.
- RAID 기법은 저용량, 저성능, 저가용성인 디스크를 배열 구조로 중복 구성함으로써 고용량, 고성능, 고가용성 디스크를 대체하고자 한다.
- 데이터 분산 저장에 의한 동시 엑세스가 가능하며, 병렬 데이터 채널에 의한 데이터 전송 시간이 단축되는 장점이 있다.

### 4.0. RAID 0(스트라이핑, Non-Redundant)

<img width="702" alt="image" src="https://github.com/user-attachments/assets/0b690296-afea-49b5-b398-8407fb36e8fe">


- 내 파일을 여러개의 디스크에 분산 저장하는 방식이다.
- 하나의 프로세서에서 처리하는 방식보다 n배(여기서는 4배) 빠르다.
- 장애에 내성이 없고, 하나의 디스크가 고장나면 데이터가 손실된다.

### 4.1. RAID 1(미러링)

<img width="733" alt="image" src="https://github.com/user-attachments/assets/03539fdf-9c79-4d89-b634-6d7955d59b11">


- 동일한 데이터를 두 개 이상의 디스크에 저장한다.
- 장애 내성이 강하지만, 비용이 많이 들고 성능 향상이 크지 않다.

### 4.2. RAID 2(Hamming Code를 활용한 방법)

<img width="721" alt="image" src="https://github.com/user-attachments/assets/875f6da0-6611-452d-a458-61d790ebe458">


- 4비트의 데이터를, 비트 단위로 4개의 디스크에 분산하여 저장하고, 3비트의 해밍코드를 생성함
- 데이터 장애가 발생해도 유추 가능함. 지금 잘 쓰이지 않음.

### 4.3. RAID 3 (Bit 단위 Parity bit)

<img width="723" alt="image" src="https://github.com/user-attachments/assets/9de16531-66a4-4813-aa66-327ce653e744">


- 1개의 패리티 비트를 이용하여 장애가 난 데이터를 유추하는 방식
- parity bit로 인해 write 성능은 하락

### 4.4. RAID 4 (Block 단위 Parity-bit)

<img width="747" alt="image" src="https://github.com/user-attachments/assets/5f89a28b-c290-4755-af9e-07a5e0aa0360">


- 

### 4.5. RAID 5 (Block 단위 Parity-bit w/ distribute)

<img width="750" alt="image" src="https://github.com/user-attachments/assets/6eb32d53-c02d-4494-9edf-6dca3901af15">


- RAID 4 방식에서, 하나에 뭉쳐있던 패리티비트를 각 디스크로 분산하는 방식
