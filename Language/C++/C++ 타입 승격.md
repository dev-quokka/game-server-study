# C++ 타입 승격 (Type Promotion)

## 타입 승격이란?

C++에서 서로 다른 타입끼리 연산할 때,  
**더 큰 범위의 타입으로 자동 변환한 후 연산을 수행하는 것**을 의미합니다.

이를 **암시적 형변환 (Implicit Casting)** 이라고 합니다.

<br>

## 타입 우선순위

숫자 타입은 아래 순서로 승격됩니다.

```
int → long long → float → double → long double
```

- 오른쪽으로 갈수록 더 큰 범위 / 더 높은 정밀도   
- 연산 시 두 타입이 다를 경우, 더 큰 타입으로 맞춘 후 계산됩니다.

<br>

## 연산 규칙

### 1. 더 큰 타입으로 변환 후 연산

```cpp
int + long long → long long
int + double → double
```
>항상 더 큰 타입 기준으로 계산

<br>

### 2. 한 번 승격되면 이후 연산도 유지

```cpp
1LL * a * b
```

> 1LL * a → long long   
> 이후 모든 연산 → long long

<br>

### 3. 정수끼리는 정수 연산

```cpp
int a = 3 / 2; // 결과: 1
```
> 소수점은 버려짐

<br>

### 4. 실수가 포함되면 실수 연산

```cpp
double a = 3 / 2.0; // 결과: 1.5
```

> 실수 연산이 수행됨

<br>

## 자주 발생하는 실수

- ### 이미 overflow 발생 후 캐스팅

```cpp
// a와 b는 int형
long long x = (long long)(a * b);
```

> a * b가 먼저 int로 계산되어 overflow 발생

<br>

#### 올바른 방법

```cpp
// a와 b는 int형
long long x = (long long)a * b;

// 또는

long long x = 1LL * a * b;
```

> 연산 전에 long long 타입을 포함시켜야   
> 전체 연산이 long long으로 수행됩니다.

<br>


- ### 나눗셈 실수

```cpp
double x = a / b;
```

> 이미 정수 나눗셈 수행됨

<br>

#### 올바른 방법

```cpp
double x = (double)a / b;
```

> 실수 타입을 먼저 포함시켜야   
> 나눗셈이 실수 연산으로 수행됩니다.


<br>

## 📌 핵심 정리

- 연산은 항상 더 큰 타입 기준으로 수행됨
- 따라서 연산 전에 타입을 올려야 안전
- 잘못된 캐스팅은 overflow를 막지 못함

<br>

> 타입 승격을 이해하면 overflow 문제를 사전에 방지할 수 있습니다.

