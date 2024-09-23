# Average Load In Linux
|                                         박정주                                         |                                      박웅빈                                      |
| :-------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------: |
| <img  width="100px" src="https://avatars.githubusercontent.com/gorapang" /> | <img width="100px" src="https://avatars.githubusercontent.com/Ungbbi" /> |
|                       [@gorapang](https://github.com/gorapang)                        |           [@Ungbbi](https://github.com/Ungbbi)           |
___
# 01. What is Avearage Load ?
시스템이 느려졌다고 느낄 때, 가장 먼저 하는일은 보통 `top`이나 `uptime` 명령어를 실행해 시스템의 부하(load)를 확인하는 것이다.

```bash
$ uptime
02:34:03 up 2 days, 20:14, 1 user, load average: 0.63, 0.83, 0.88
```
위 `uptime`명령어의 출력 결과를 해석해보면</br>
- `02:34:03`  : 현재 시간
- `up 2 days, 20:14` : 시스템 가동 시간
- `1 user` : 로그인된 사용자 수
- `load average : 0.63, 0.83, 0.88` : 각각 최근 1분, 5분, 15분 동안의 평균 부하
___
# 02. load average(평균부하)
- **실행 가능한 상태**(Runnable), **중단 불가능한 상태**(Uninterruptible)에 있는 **프로세스들의 평균 수** 이다.
- **Active process**들의 평균 수 이다. 
- 단위 시간당 CPU 사용률이 **아니다**.
- -> **(단위 시간당 Active process의 평균 수 이다.)**

</br>
그렇다면 Linux에서 Runnable/Uninterruptible process는 무엇일까?
</br>

  ### ✅ Runnable process
  - Runnable process는 CPU를 사용 중이거나 CPU 시간을 기다리고 있는 프로세스다.
  - `ps` 명령에서 **R 상태**로 표시되는 프로세스들이다.
  
  ### ✅ Uninterruptible process
  - 중요한 커널 프로세스 내에 있어 중단 불가능한 상태의 프로세스이다.
  - `ps`명령에서 **D 상태**로 표시되는 프로세스들이다.
  - Ex) 하드웨어 장치로부터 I/O를 기다리는 프로세스

  #### Q1) 만약, Average load가 2라면 무엇을 의미할까?
  - CPU : 2 -> 모든 CPU가 완전히 점유된 상태
  - CPU : 4 -> CPU 용령의 50%가 여유 있는 상태
  - CPU : 1 -> 절반의 Process가 CPU 시간을 놓고 경쟁하고 있는 상태

___
## 그렇다면 적절한 load average는 무엇일까?
처음 `uptime` 명령어의 출력에서, 어떤 범위의 load average가 System load가 높은지, 낮은지 판단을 어떻게 할까?
</br>
### ✅ CPU 수
- 이상적인 load average는 **CPU 수**와 같을 때이다. </br> 그러므로 우선 **CPU 개수**를 아는 것이 첫번째이다.
- CPU 개수를 알고 싶다면 `top` 명령어나 `/proc/cpuinfo` 파일을 읽어오면 된다.

- ```bash
  $ grep 'model name' /proc/cpuinfo | wc -l
  2
  ```

### ✅ load average 값
- `uptime` 명령어를 통해 나온 값들 중 load average를 살펴보자</br>

- 부하가 감소한 경우</br>
  `load average: 0.63, 0.83, 0.88`</br>
  15분일 때가 가장 값이 높았으나 점차 최근일 수록 값이 낮아진 것을 보아 부하가 점점 감소했음을 알 수 있다.

  
- 부하가 증가한 경우 </br>
  `load average: 0.89, 0.60, 0.55`</br>
  5분 전까지만 해도 부하가 없었으나 1분 사이에 급격히 부하가 증가했음을 알 수 있다.

### ✅ load average 와 CPU 사용률
앞서 적어놓았듯이 load average는 Runnable, Uninterruptible 상태의 process 수를 나타내지만,</br>CPU 사용률은 CPU가 실제로 얼만큼 바빴는지를 보여준다.
- 따라서 I/O를 기다리는 프로세스가 많으면 **average load는 높을 수 있으나 CPU 사용률은 낮을 수 있다**.

___
# 03. Case Study
세 가지 예시를 통해 위의 세 가지 상황을 이해하고, `iostat`, `mpstat`, `pidstat` 등의 도구를 사용하여 평균 부하 증가의 원인을 파악해보자.

- `stress` :
    - 리눅스 시스템 부하 테스트 도구
    - 비정상적인 프로세스로 인해 평균 부하가 증가하는 시나리오를 시뮬레이션하는 데 사용
    
- `sysstat`:
    - 시스템 성능을 모니터링하고 분석하는 데 사용되는 리눅스 성능 도구들을 포함한다.
    - `mpstat`: 멀티 코어 CPU 성능을 분석하는 데 사용하는 도구로, 각 CPU 및 모든 CPU에 대한 실시간 성능 메트릭을 확인할 수 있다.
    - `pidstat`: 프로세스 성능을 분석하는 데 사용하는 도구로, 프로세스별 CPU, 메모리, I/O, 컨텍스트 스위치의 실시간 성능 메트릭을 확인할 수 있다.

- 실험환경
    - Ubuntu 22.04
    - Machine configuration: 2 CPUs, 8GB RAM
- 설치

```bash
sudo apt install stress 
sudo apt install sysstat
```

실험을 위해 3개의 터미널을 열어 동일한 유저로 로그인한다.

## **Scenario 1: CPU-intensive process**

터미널 1️⃣

- `stress`명령어를 통해 10분간 CPU를 100%로 사용하는 프로세스를 실행

```bash

stress --cpu 1 --timeout 600
```

터미널 2️⃣

- `uptime` 명령어로 시스템 부하를 확인하면, CPU 부하가 점차 1.00으로 증가하는 것을 볼 수 있다.

```bash
watch -d uptime
```

터미널 3️⃣

- `mpstat` 명령어로 CPU 사용률을 실시간으로 모니터링

```bash
mpstat -P ALL 5
```

```bash
02:13:02 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
02:13:07 PM  all   49.95    0.00    0.00    0.00    0.00    0.10    0.00    0.00    0.00   49.95
02:13:07 PM    0    0.00    0.00    0.00    0.00    0.00    0.20    0.00    0.00    0.00   99.80
02:13:07 PM    1  **100.00**    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00
```

- 특정 프로세스가 CPU를 100% 사용하고 있음을 보여준다.

- `pidstat`명령어를 사용하여 CPU 사용률이 100%인 프로세스가 무엇인지 확인할 수 있다.
    - `stress` 프로세스가 CPU 사용률 100%를 차지하고 있음

```bash
username@servername:~$ pidstat -u 5 1
Linux 5.15.0-122-generic (servername)   09/23/2024      _x86_64_        (2 CPU)

02:14:39 PM   UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
02:14:44 PM     0       229    0.00    0.20    0.00    0.00    0.20     1  irq/18-vmwgfx
02:14:44 PM  1000      6394  100.00    0.00    0.00    0.00  100.00     1  stress
```

## **Scenario 2: I/O-intensive process**

터미널 :one:

- I/O 부하를 시뮬레이션하기 위해, `stress` 명령어로 I/O 작업을 계속 실행한다.

```bash
stress -i 1 --timeout 600
```

터미널 2️⃣

- `uptime`을 실행하면 부하가 증가하는 것을 볼 수 있다.

```bash
watch -d uptime
```

터미널 3️⃣

- `mpstat`를 실행하면 I/O 대기 시간(iowait)이 크게 증가한 것을 확인할 수 있다.

```bash
mpstat -P ALL 5
```

![image](https://github.com/user-attachments/assets/c8514b97-7016-4436-8a23-81f8a476dab6)


## **Scenario 3: 많은 수의 프로세스**

시스템의 CPU 수보다 많은 프로세스를 실행하면 과부하가 발생한다. 

- `stress`로 8개의 CPU 집약적인 프로세스를 실행한다.

```bash
stress -c 8 --timeout 600
```

- CPU가 2개인 시스템에서는 평균 부하가 7.97까지 상승한다.

2개의 CPU만 존재하는 시스템에서, 8개의 CPU-intensive 프로세스가 실행된다. 동시에 실행 중인 8개의 프로세스가 CPU를 할당받기를 기다리게 된다. 즉, CPU가 동시에 처리할 수 있는 작업 수보다 많은 작업이 쌓여서 과부하가 발생한다.

---

# Summary

**평균 부하**는 시스템의 전체 성능을 평가하기 위한 방법으로, 시스템의 전반적인 부하 상황을 나타낸다. 그러나 평균 부하만을 보고서는 어디에서 병목 현상이 발생하는지 직접적으로 알 수 없다. 따라서  평균 부하를 이해할 때 다음 사항들을 고려해야 한다:

1. 높은 평균 부하는 CPU-intensive 프로세스에 의해 발생할 가능성이 크다.
2. 평균 부하가 높다고 반드시 CPU 사용률이 높은 것은 아니며, I/O 증가로 인해 평균 부하가 증가할 수도 있다.
3. 높은 부하를 발견했을 때, `mpstat`, `pidstat`와 같은 도구를 사용하여 부하의 원인을 분석해야 한다.
