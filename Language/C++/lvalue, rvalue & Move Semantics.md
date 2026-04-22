# lvalue, rvalue & Move Semantics

## lvalue, rvalue 기본 개념

| 구분 | lvalue                                                | rvalue                              |
| ---- | ----------------------------------------------------- | ----------------------------------- |
| 정의 | 메모리 상에 **식별 가능한 위치(주소)** 를 가진 표현식 | **일시적인 값**. 식이 끝나면 사라짐 |
| 특징 | 이름이 있고, 재사용 가능, `&`로 주소 획득 가능        | 이름 없음, 주소 획득 불가           |

<br>

### 가장 쉬운 판별법

> 식별자(이름)를 가지고 있어서, `&` 연산자(주소 연산자)를 붙일 수 있으면 lvalue, 없으면 rvalue

```cpp
int x = 10;
&x;        // 가능 → x는 lvalue
&10;       // 컴파일 에러 → 10은 rvalue
&(x + 1);  // 에러 → x+1 결과는 rvalue
```

<br>

## 이름의 유래

대입 연산자 `=`을 기준으로

- 왼쪽(left)에 올 수 있는 값 → **lvalue**
- 오른쪽(right)에만 올 수 있는 값 → **rvalue**

```cpp
int a = 5;    // a: lvalue, 5: rvalue
5 = a;        // 에러 → rvalue는 좌변에 못 옴
a = a + 1;    // 가능 → a+1은 rvalue지만 우변이라 OK
```

> 단, 이건 유래일 뿐 완벽한 규칙은 아닙니다.
> const int c = 5;에서 c는 lvalue지만 좌변에 올 수 없습니다.

<br>

## lvalue, rvalue 예시

### lvalue인 것들

- 변수 이름 (x, arr[0])
- 역참조 `*p`
- 전위 증감 `++x`
- 함수가 `T&`를 반환하는 경우
- 문자열 리터럴 ("hello")

### rvalue인 것들

- 리터럴 `10`, `3.14`, `true`
- 산술 연산 결과 `x + 1`
- 후위 증감 `x++` (증가 전 값의 복사본)
- 함수가 `T`를 값으로 반환하는 경우

<br>

```cpp
int x = 10;
int* p = &x;

x;        // lvalue
10;       // rvalue
x + 1;    // rvalue
++x;      // lvalue
x++;      // rvalue
*p;       // lvalue
```

<br>

## rvalue reference (&&)

C++11에서 **rvalue를 참조할 수 있는 새로운 참조 타입**이 추가됬습니다.

```cpp
int x = 10;

int&  lref  = x;      // lvalue reference (기존)
int&& rref  = 10;     // rvalue reference (신규)
int&& rref2 = x;      // 에러! x는 lvalue
int&& rref3 = x + 1;  // OK, x+1은 rvalue
```

<br>
 
#### 중요 포인트
 
int&& rref = 10; 에서 **rref 자체는 lvalue** 입니다.
 
- 이름이 있고, 주소를 얻을 수 있기 때문
- 타입은 rvalue reference지만, **표현식으로서는 lvalue**

> 이름이 있으면 lvalue 규칙을 꼭 기억할 것!

---

#### 🔹 &&는 참조의 참조가 아니다

**`&&`는 `&`를 두 번 쓴 게 아니라, 하나의 독립된 기호**입니다.

```cpp
int&  lref;   // lvalue reference
int&& rref;   // rvalue reference (&&가 하나의 토큰)
```

---

<br>

## Move Semantics (이동 시맨틱)

rvalue reference의 진짜 목적은 **불필요한 복사 제거**입니다.

```cpp
std::vector<int> MV() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    return v;  // 임시 객체(rvalue)를 반환
}
```

#### 🔹 C++11 이전의 문제

MV()가 반환한 임시 벡터는 곧 소멸될 rvalue입니다.  
그런데도 **깊은 복사(deep copy)** 로 모든 원소를 전부 새 메모리에 복제한 뒤, 원본은 그대로 버려집니다.

> 소멸될 객체의 리소스를 복사하여 사용한 뒤 원본을 버리는 꼴 → **낭비**

#### 🔹 C++11 이후의 해법

> 어차피 사라질 rvalue라면, 내부 데이터(포인터)를 훔쳐오자 → 이것이 **이동(move)**

<br>

#### 핵심 아이디어

**rvalue(곧 사라질 객체)** 에 한해서, 새 메모리를 할당하지 말고 **포인터 소유권만 이전**하는 것

이를 수행하는 특별한 생성자가 **이동 생성자(move constructor)** 이고,  
&&(rvalue reference)를 매개변수로 받도록 만듭니다.

<br>

```cpp
class CM {
    char* data;
    size_t size;

public:
    // 복사 생성자 — 새 메모리 할당 + 전체 복사 (느림)
    CM(const CM& other);

    // 이동 생성자 — 포인터만 훔쳐옴 (빠름)
    CM(CM&& other) noexcept
        : data(other.data), size(other.size) {
        other.data = nullptr;  // 원본 무효화 (double free 방지)
        other.size = 0;
    }

    ~CM() { delete[] data; }
};
```

<br>

### 주의할 점

#### 1. 원본을 `nullptr`로 만드는 이유는?

포인터만 복사하면 두 객체가 **같은 메모리**를 가리킵니다.  
그대로 두면 소멸자가 `delete[]`를 두 번 호출 → **이중 해제(double free)** 로 크래시 발생.

> 원본을 `nullptr`로 만들면 `delete[] nullptr`은 안전하게 아무 일도 하지 않습니다.

<br>
 
#### 2. `noexcept`를 꼭 붙이기
 
std::vector 같은 표준 컨테이너는 재할당 시  
**이동 생성자가 `noexcept`가 아니면 안전을 위해 복사로 폴백**합니다.

> 성능을 챙기려면 반드시 `noexcept` 명시하자!

<br>

## std::move

이동 생성자는 **rvalue를 받을 때만 호출**됩니다.  
그런데 lvalue를 더 이상 사용하지 않아 내용물을 **강제로 이동**시키고 싶을 때가 있습니다.  
그럴 때 쓰는 것이 바로 **std::move** 입니다.

단, lvalue를 rvalue로 **캐스팅**해주는 함수일 뿐  
std::move는 **이동시키는 함수가 아닙니다.**

```cpp
std::string s1 = "hello";
std::string s2 = s1;             // 복사 생성자 호출 (s1은 lvalue)
std::string s3 = std::move(s1);  // 이동 생성자 호출 (s1을 rvalue로 캐스팅)
// 이후 s1은 유효하지만 명시되지 않은 상태. 접근은 가능하나 내용은 보장 안 됨
```

<br>

### std::move 동작 흐름 정리

1. std::move(s1) → s1을 rvalue reference(std::string&&)로 **캐스팅**
2. s3의 이동 생성자가 호출됨 (rvalue를 받기 때문)
3. 이동 생성자가 s1의 내부 포인터를 훔쳐옴
4. s1은 내용이 비워진 상태로 남음

<br>

### 주의할 점

#### 1. std::move 후 원본(s1)은 **읽지 말기!**

유효한 상태이긴 하지만 어떤 값을 가질지 보장되지 않습니다.  
(재할당은 가능 → s1 = "새로운 값" 은 가능)

<br>

#### 2. const 객체에 std::move를 쓰면 이동이 아니라 **복사**가 일어납니다.

```cpp
const std::string a = "aaa";
std::string b = "bbb";

std::string ta = std::move(a);  // copy 발생
std::string tb = std::move(b);  // move 발생

std::cout << a.size() << '\n';  // 3 (원본 유지)
std::cout << b.size() << '\n';  // 구현에 따라 다름 (보통 0)
```

<br>

#### 3. 기본 타입(int, double, raw pointer 등)에 std::move는 효과가 없습니다.

훔쳐올 자원이 없어 이동과 복사가 동일하게 동작하며, 원본 값도 그대로 유지됩니다.

<br>

## 📌 핵심 정리

- lvalue는 식별 가능한 객체이며, 주소를 가질 수 있음
- rvalue는 일시적인 값으로, 대부분 표현식 종료 시 소멸
- rvalue reference(`&&`)는 rvalue를 참조하기 위해 도입됨
- std::move는 lvalue를 rvalue로 캐스팅하는 함수
- Move Semantics는 복사 대신 자원 소유권을 이전하여 성능을 향상시킴

<br>

> **lvalue는 이름 있는 것, rvalue는 곧 사라질 것입니다.**
> **rvalue reference(`&&`)와 `std::move`는 이를 활용해 복사 대신 이동으로 성능을 챙기는 C++11의 설계입니다.**
