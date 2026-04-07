# Vector Container

## Vector란?

vector는 C++ 표준 라이브러리(STL)에서 제공하는  
**동적 배열(Dynamic Array) 기반 컨테이너**입니다.

연속된 메모리 공간을 사용하며,  
필요에 따라 크기가 자동으로 증가합니다.

배열과 유사하지만 크기를 동적으로 조절할 수 있다는 점이 특징입니다.

<br>

## Vector Container 구조

<img src="../C++/images/벡터 구조.png" width="520" height="304"/>

<br>

- begin(): 첫 번째 요소를 가리킴
- end(): 마지막 요소 다음 위치를 가리킴
- push_back(): 뒤쪽에 요소 추가
- pop_back(): 뒤쪽 요소 제거
- insert(): 중간 위치에 삽입 → 이후 요소 이동 발생

<br>

## 특징

- 연속된 메모리 구조 (배열 기반)
- 인덱스를 통한 빠른 접근 (O(1))
- 뒤쪽 삽입/삭제는 빠름
- 중간 삽입/삭제는 느림 (O(n))

> 앞쪽 삽입/삭제도 가능하지만, 모든 요소를 이동해야 하므로 비효율적입니다.

---

## 시간복잡도 (Time Complexity)

| Operation       | Time Complexity |
| --------------- | --------------- |
| Access          | O(1)            |
| Push Back       | O(1)            |
| Insert (middle) | O(n)            |
| Delete (middle) | O(n)            |

---

<br>

## Vector 사용법

### 🔹 선언 및 초기화

```cpp
vector<int> v;            // 빈 vector
vector<int> v(10);        // 크기 10 (0으로 초기화)
vector<int> v(10, 2);     // 크기 10, 값 2로 초기화
vector<int> v = {1,2,3};  // 초기값 설정
vector<int> v2(v1);       // 복사 생성
```

<br>

### 🔹 요소 추가 / 삭제

```cpp
push_back()  // 마지막에 요소 추가
pop_back()   // 마지막 요소 삭제

insert() // 중간 삽입 → O(n)
erase() // 중간 삭제 → O(n)
```

> 중간 삽입/삭제는 모든 요소를 이동해야 하므로 비효율적
> 이런 경우 다른 자료구조 사용 권장

<br>

### 🔹 size & capacity

- size(): 컨테이너에 저장된 원소의 개수
- capacity(): 실제로 할당된 메모리 공간의 크기

```cpp
vector<int> v;

v.push_back(1);
v.push_back(2);
v.push_back(3);

cout << v.size();     // 3
cout << v.capacity(); // 4
```

> vector는 capacity가 부족해지면  
> 일반적으로 **기존 크기의 2배로 확장**됩니다.

<br>

### 🔹resize vs reserve

- resize(n): size 변경 (데이터 생성/삭제)
- reserve(n): capacity 확보 (재할당 방지)

```
v.resize(2);   // size만 줄어듦 (capacity 유지)
v.reserve(10); // capacity만 증가
```

> 데이터 개수를 미리 알고 있다면 reserve() 사용이 성능에 유리

<br>

## 장점

- 인덱스를 통한 빠른 접근 (O(1))
- 연속된 메모리 구조로 캐시 효율이 높음
- 동적 크기 관리 가능
- 구조가 단순하여 사용이 쉬움

<br>

## 단점

- 중간 삽입/삭제 비효율적 (O(n))
- capacity 증가 시 재할당 비용 발생
- 재할당 시 기존 데이터 복사 비용 발생

<br>

## 게임 서버에서의 활용

vector는 빠른 접근과 캐시 효율이 중요하기 때문에
게임 서버에서 자주 사용되는 컨테이너입니다.

- 유저 세션을 IDX 기반으로 관리
- 빠른 조회가 필요한 데이터 저장

> 대부분의 경우 vector를 기본 컨테이너로 사용하고
> 특정 상황에서만 다른 자료구조를 선택합니다.

<br>

## 📌 핵심 정리

- vector는 동적 배열 기반 컨테이너
- 연속된 메모리 구조로 O(1) 접근 가능
- push_back은 빠르지만, 중간 삽입/삭제는 느림
- capacity 증가 시 재할당 비용 발생

<br>

> 대부분의 상황에서 기본적으로 사용되는 자료구조
