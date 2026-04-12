# C++ 캐스팅 (Type Casting)

## 캐스팅이란?

캐스팅은 **데이터의 타입을 다른 타입으로 변환하는 것**을 의미합니다.

C++에서는 명확하고 안전한 타입 변환을 위해  
여러 가지 캐스팅 연산자를 제공합니다.

<br>

## C 스타일 vs C++ 스타일 캐스팅

### C 스타일 캐스팅

```cpp
(int)a;
```

- 하나의 문법으로 다양한 변환 수행
- 내부적으로 여러 캐스팅이 혼합되어 동작
- 타입 안정성이 낮고 의도가 명확하지 않음

<br>

### C++ 스타일 캐스팅

```cpp
static_cast<int>(a);
```

- 캐스팅 목적이 명확하게 구분됨
- 컴파일 단계에서 타입 체크 가능
- 코드 가독성과 안정성 향상

<br>

> C++에서는 C 스타일 캐스팅보다  
> C++ 스타일 캐스팅 사용이 권장됩니다.

<br>

## C++ 캐스팅 종류

### 1. static_cast

```cpp
int a = 10;
double b = static_cast<double>(a);
```

- 기본 타입 변환 (int → double)
- 상속 관계에서 업캐스팅
- 컴파일 타임에 타입 체크 수행

> 일반적인 타입 변환에서 가장 많이 사용되는 캐스팅

<br>

### 2. dynamic_cast

```cpp
Base* b = new Derived();
Derived* d = dynamic_cast<Derived*>(b);
```

- 다형성 환경에서 안전한 다운캐스팅
- 런타임 타입 체크 수행 (RTTI)
- 실패 시 nullptr 반환 (포인터 기준)

> 부모 → 자식 변환 시 안전성을 보장할 때 사용하는 캐스팅

<br>

### 3. const_cast

```cpp
const int a = 10;
int* b = const_cast<int*>(&a);
```

- const / volatile 속성 제거 또는 추가
- 실제 데이터 변경은 주의 필요

> const 제거가 필요한 특수한 경우에만 사용

<br>

### 4. reinterpret_cast

```cpp
int a = 10;
char* p = reinterpret_cast<char*>(&a);
```

- 포인터를 다른 타입으로 재해석
- 메모리를 그대로 다른 타입으로 해석

> 저수준 캐스팅으로 사용 시 주의 필요

<br>

## 언제 어떤 캐스팅을 사용할까요?

- 일반적인 타입 변환 → static_cast
- 다형성 기반 다운캐스팅 → dynamic_cast
- const 속성 제거 → const_cast
- 메모리 / 네트워크 처리 → reinterpret_cast

<br>

## 📌 핵심 정리

- C++에서는 목적에 맞는 캐스팅을 명확히 사용해야 함
- static_cast는 일반적인 변환에서 기본적으로 사용
- dynamic_cast는 안전한 다운캐스팅을 보장
- reinterpret_cast는 저수준 작업에서만 제한적으로 사용

<br>

> 명확한 캐스팅 사용은 코드의 안정성과 가독성을 높일 수 있습니다.
