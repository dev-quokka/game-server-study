# C++ 키워드 레퍼런스

신입 개발자가 코드에서 자주 마주치지만 의미가 헷갈리는 키워드들을 정리한 문서입니다.

<br>

---

## explicit

생성자의 암묵적 변환을 막는 함수

<br>

### 언제 사용하나요?

- 단일 인자 생성자를 만들 때 거의 항상
- 의도치 않은 타입 변환으로 생기는 버그 방지

<br>

### 코드 예시

```cpp
class Foo {
public:
    explicit Foo(int x) {}
};
void f(Foo foo) {}

f(42);        // 에러 (explicit 덕분에 막힘)
f(Foo(42));   // 명시적이면 OK
```

---

## inline

여러 번 정의돼도 괜찮다는 표시 (성능 인라인화 X)

<br>

### 언제 사용하나요?

- 헤더 파일에 함수/변수 정의를 둘 때 (ODR 위반 방지)
- C++17부터는 inline 변수도 가능
- 최적화 관점의 "인라인 확장"은 **현대 컴파일러가 알아서 함**. inline 키워드와 거의 무관.

<br>

> 'inline = 함수를 인라인 확장한다'는 옛날 의미  
> 현재는 **ODR 면제** 용도가 더 중요함

---

## noexcept

이 함수는 예외를 던지지 않는다고 선언

<br>

### 언제 사용하나요?

- 이동 생성자 / 이동 대입 연산자 (**매우 중요**)
- 소멸자 (기본으로 noexcept)
- swap 함수
- 예외를 절대 던질 수 없는 함수

<br>

### 왜 이동 생성자에서 중요한가요?

std::vector가 재할당할 때, 원소의 이동 생성자가 noexcept가 아니면  
안전을 위해 copy로 대체하여 성능 차이가 큼.

```cpp
class MyClass {
public:
    MyClass(MyClass&& other) noexcept { ... } // 권장되는 방법
};
```

---

## constexpr

컴파일 타임에 계산 가능한 상수/함수임을 표시

<br>

### 언제 사용하나요?

- 컴파일 타임에 값이 결정되는 상수
- 템플릿 인자로 쓸 값 계산
- 성능이 중요한 상수 표현식

<br>

### const와 차이

- const: 런타임 상수 (변경 불가)
- constexpr: 컴파일 타임 상수 (가능하면 컴파일 시 계산)

```cpp
constexpr int square(int x) { return x * x; }
constexpr int val = square(5);  // 컴파일 타임에 25로 계산됨

int arr[square(3)];  // 배열 크기로 사용 가능
```

---

## static

맥락에 따라 의미가 다른 다의어. '유지' 또는 '공유'로 이해

<br>

### 사용법

- 함수 내 static 변수: 프로그램 끝까지 값이 유지됨 (함수 종료해도 안 사라짐)
- 클래스 멤버 static: 클래스 전체에 하나만 존재, 모든 인스턴스가 공유
- 전역 static: 해당 파일 안에서만 사용 가능 (현대는 익명 namespace 선호)

<br>

### 코드 예시

```cpp
void count() {
    static int n = 0;  // 호출 간에 값 유지
    n++;
}

class Foo {
    static int total;  // 모든 Foo 객체가 공유
};
```

<br>

### 주의할 점

> 이름만 같을 뿐 세 용법의 의미가 다름. 맥락으로 판단.

---

## mutable

const 객체에서도 **수정 가능한** 멤버 변수 표시

<br>

### 언제 사용하나요?

- 논리적으론 const지만 내부 상태는 바꿔야 할 때
- 캐싱, mutex, 지연 계산 등
- 람다에서 캡처한 값을 수정하고 싶을 때

<br>

### 코드 예시

```cpp
class Database {
    mutable int cache = -1;  // const 메서드에서도 수정 가능
public:
    int query() const {
        if (cache == -1) cache = compute();
        return cache;
    }
};
```

<br>

### 주의할 점

> 남용하면 const의 의미가 무너짐. 꼭 필요할 때만 사용.

<br>

### mutable vs const_cast

같은 목적으로 쓰이지만 성격이 완전히 다릅니다.

- mutable: **설계 의도** ('이 멤버는 원래 바뀔 수 있음'을 선언) → 안전
- const_cast: **회피 수단** (const 약속을 억지로 벗겨냄) → 원본이 진짜 const면 UB

> 내가 설계하는 클래스라면 **mutable이 정답** (const_cast는 최후의 수단)

---

## nullptr (NULL과 비교)

타입 안전한 널 포인터. C++11부터 NULL 대신 써야 함.

<br>

### 왜 NULL이 문제인가요?

NULL은 보통 그냥 0(int)이라 오버로드가 꼬일 수 있습니다.

<br>

### 코드 예시

```cpp
void f(int x)    {}
void f(char* p)  {}

f(NULL);     // int 오버로드 선택됨 (의도와 다름)
f(nullptr);  // pointer 오버로드 선택됌
```

> C++11 이상에서는 **nullptr만 사용**. NULL은 쓰지 말기.

---

## using vs typedef

둘 다 타입 별칭을 만듦. **using이 현대적**이고 더 강력.

<br>

### 왜 using이 나은 가요?

- 읽기 쉬움 (특히 함수 포인터)
- **템플릿 별칭 지원** (typedef는 불가능)

<br>

### 코드 예시

```cpp
// 기본 용법 (동일)
typedef unsigned int uint;
using uint = unsigned int;  // 권장
```

> C++11 이상에서는 **using 기본 사용**

---

## volatile

이 변수는 예상 밖으로 바뀔 수 있으니 최적화 금지 알림

<br>

### 언제 사용하나요?

- 하드웨어 레지스터 접근 (임베디드)
- 메모리 매핑 I/O
- 일반 애플리케이션에서는 **거의 쓸 일 없음**

<br>

### 주의할 점

> **volatile은 스레드 동기화 용도가 아님!**  
> 스레드 간 공유는 std::atomic이나 std::mutex 사용

| 용도          | 사용 도구               |
| ------------- | ----------------------- |
| 하드웨어 접근 | volatile                |
| 스레드 동기화 | std::atomic, std::mutex |

---

## thread_local

스레드마다 **독립적인 인스턴스**를 가지는 변수.

<br>

### 언제 사용하나요?

- 스레드별 캐시 / 난수 생성기 / 임시 버퍼
- 동기화 없이 스레드별 상태를 유지하고 싶을 때

<br>

### 코드 예시

```cpp
thread_local int counter = 0;

// 각 스레드의 counter는 서로 독립적 (동기화 불필요)
```

<br>

### static과 비교

- static: 프로그램 전체에 하나 (스레드 공유 → 동기화 필요)
- thread_local: 스레드마다 하나 (동기화 불필요)

---

<br>

## 요약

| 키워드       | 용도                  | 꼭 기억할 것                 |
| ------------ | --------------------- | ---------------------------- |
| explicit     | 암묵적 변환 방지      | 단일 인자 생성자엔 거의 항상 |
| inline       | ODR 면제              | 성능 인라인 아님             |
| noexcept     | 예외 안 던짐 선언     | 이동 생성자엔 필수           |
| constexpr    | 컴파일 타임 계산      | const와 구분                 |
| static       | 유지/공유             | 맥락별 의미 다름             |
| mutable      | const에서도 수정 가능 | 남용 금지                    |
| nullptr      | 타입 안전 널 포인터   | NULL 쓰지 말기               |
| using        | 타입 별칭             | typedef보다 현대적           |
| volatile     | 최적화 억제           | 동기화 도구 아님             |
| thread_local | 스레드별 저장소       | 스레드 독립 변수             |
