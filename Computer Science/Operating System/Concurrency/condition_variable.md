# Condition Variable

<br>

condition_variable은 멀티스레드 환경에서 특정 조건이 만족될 때까지 스레드를 **효율적으로 대기**시키는 동기화 도구입니다.

다른 스레드가 **공유 변수를 수정**하고 **condition_variable로 통지**할 때까지

스레드나 여러 스레드를 대기하도록 하는데 사용할 수 있는 동기화 기법입니다.

언제나 뮤텍스와 연동되어서 동작하여 스레드에 안전하게 동작합니다.

<br>

## Condition Variable은 왜 필요한가? (busy-wait 문제)

멀티스레드 환경에서 특정 조건이 만족될 때까지 대기해야 하는 상황은 자주 발생합니다.

이때 condition variable 없이 조건을 기다리면 **busy-wait** 방식을 사용하게 됩니다.

<br>

| 방식 | CPU 사용률 | 문제점 |
|------|-----------|--------|
| busy-wait | 계속 100% 유지 | CPU 자원 낭비 |
| condition_variable | 대기 중 0% | 없음 |

<br>

- busy-wait: 조건이 될 때까지 CPU를 계속 소모

<br>


## 멤버 함수

#### wait()
조건이 만족될 때까지 스레드를 sleep 상태로 대기시킵니다.

```cpp
cv.wait(lock, []{ return dataReady; }); // 조건 불만족 시 sleep
```


내부 동작 흐름은 다음과 같다.

```
1. mutex unlock       → 다른 스레드가 접근할 수 있게 풀어줌
2. 스레드 sleep       → CPU 사용 없이 대기
3. notify 신호 수신
4. mutex lock         → 다시 잠금
5. 조건 확인
6-1. 조건 불만족      → 다시 1번으로 (spurious wakeup 대비)
6-2. 조건 만족        → 진행
```


<br>

#### notify_one()
대기 중인 스레드 중 하나를 깨웁니다. OS 스케줄러가 결정하며 우선순위는 보장하지 않습니다.

```cpp
cv.notify_one();
```

<br>

#### notify_all()
notify_one()과 동일하게 동작하며 차이점은 condition variable에 대해서 대기하고 있는 모든 스레드를 깨웁니다.

```cpp
cv.notify_all();
```

| | notify_one | notify_all |
|--|-----------|------------|
| 깨우는 스레드 수 | 1개 | 전부 |
| 사용 상황 | 작업 큐, 매칭 시스템 | 종료 신호, 상태 변경 브로드캐스트 |

<br>

## spurious wakeup

spurious wakeup이란 비정상적으로 깨어난 조건 변수 알림을 뜻합니다.

즉, notify가 없었는데도 OS가 스레드를 깨우는 현상입니다.

조건을 같이 넣으면 spurious wakeup으로 깨어나도 조건이 불만족이면 다시 sleep 상태로 돌아갑니다.

```cpp
// 조건 없이 사용하면 spurious wakeup에 취약
cv.wait(lock);

// 반드시 조건을 함께 넣어야 안전
cv.wait(lock, []{ return dataReady; });
```

<br>

### spurious wakeup 발생 이유

일반적으로 condition_variable을 통해서 notify 함수를 호출 할 때 mutex의 잠금을 유지할 필요가 없습니다. 

그렇기 때문에 아래와 같은 시나리오가 발생할 수 있습니다. 

```
스레드 A   : condition variable을 통해서 queue에 들어온 데이터 처리 대기
스레드 B, C: 데이터 추가 후 통지 알림을 보내는 스레드

1. 스레드 B가 queue에 데이터를 추가한 후에 mutex를 해제한 후 통지 알림을 보내기 전에 **컨텍스트 스위치** 발생
2. 스레드 C가 queue에 데이터를 추가한 후에 통지 알림을 보냄
3. 스레드 A는 통지를 전달받고 queue에 있는 2개의 데이터를 모두 처리
4. 스레드 B가 다시 시작되고 통지 알림을 전달
5. 스레드 A는 깨어나지만 큐가 비어 있는 상태 → spurious wakeup 발생
```

이 방식 외에 다양한 이유로 해당 현상이 발생할 수 있습니다.


<br>

## unique_lock이 필요한 이유

condition_variable의 wait()는 내부적으로 **unlock → sleep → lock**을 반복하기 때문에 중간에 unlock/lock 제어가 가능한 **unique_lock**이 필요합니다.

```cpp
std::lock_guard<std::mutex> lock(mtx);   // cv.wait()에 사용 불가

std::unique_lock<std::mutex> lock(mtx);  // cv.wait()에 사용 가능
lock.unlock(); // 명시적 unlock 가능
lock.lock();   // 명시적 lock 가능
```

notify sender는 **lock → 작업 → unlock**이 한 번씩만 수행되므로 lock_guard로 충분하다.

```cpp
// notify sender
void sender() {
    std::lock_guard<std::mutex> lock(mtx);
    dataReady = true;
    cv.notify_one();
}

// notify receiver
void receiver() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, []{ return dataReady; });
}
```

<br>

## 구현 예시

#### busy-wait (condition_variable 미사용)

```cpp
#include <iostream>
#include <thread>

bool dataReady = false;

void receiver() {
    while (!dataReady) {} // CPU 100% 낭비
    std::cout << "Data received!" << std::endl;
}

void sender() {
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    dataReady = true;
}

int main() {
    std::thread t1(receiver);
    std::thread t2(sender);

    t1.join();
    t2.join();

    return 0;
}
```

<br>

#### condition_variable 사용

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool dataReady = false;

void receiver() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, []{ return dataReady; }); // 조건 만족까지 sleep
    std::cout << "Data received!" << std::endl;
}

void sender() {
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    {
        std::lock_guard<std::mutex> lock(mtx);
        dataReady = true;
    }
    cv.notify_one();
}

int main() {
    std::thread t1(receiver);
    std::thread t2(sender);

    t1.join();
    t2.join();

    return 0;
}
```

<br>

## 게임 서버에서의 활용

게임 서버에서는 **"무언가를 기다렸다가 신호 오면 처리"** 하는 패턴이 많아 condition_variable이 자주 사용됩니다.

```
작업 큐 (Job Queue)
└── 워커 스레드: 처리할 작업 생길 때까지 cv.wait()로 대기
└── 메인 스레드: 새 패킷/이벤트 들어오면 notify_one()

플레이어 매칭 시스템
└── 매칭 스레드: 인원이 찰 때까지 cv.wait()로 대기
└── 입장 처리: 플레이어 들어올 때마다 notify_one()
```

notify_one vs notify_all 선택 기준

```
작업 1건 처리 → notify_one
  → 워커 1개만 깨워서 처리, 나머지는 계속 sleep

서버 종료 신호, 설정 변경 등 → notify_all
  → 모든 스레드에 알려야 할 때 사용
  → thundering herd 주의
```