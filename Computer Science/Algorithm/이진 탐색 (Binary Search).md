# 이진 탐색 (Binary Search)

## 이진 탐색 (Binary Search)이란?

정렬된 배열에서 원하는 값을 O(log N)에 찾는 알고리즘입니다.  
사용을 위해서는 배열이 **오름차순 정렬**되어 있어야 합니다.

<br>

## 탐색 과정

```
int s;      // 시작 인덱스
int e;      // 끝 인덱스
int mid;    // 중간 인덱스 (s + e) / 2
int target; // 찾고자 하는 값
vector<int> v // 정렬된 배열
```

1. 중간값 mid가 target과 같은지 확인합니다.
2. target과 같다면 true를 반환합니다.
3. target보다 mid값이 작다면(v[mid] < target), (mid + 1 ~ e)로 다시 검사를 진행합니다.
4. target보다 mid값이 크다면(v[mid] > target), (s ~ mid - 1)로 다시 검사를 진행합니다.

> 1~4번 과정을 반복하다
> 더 이상 검사할 곳이 없다면(s > e) 탐색 실패를 반환합니다.

<br>

### 시간 복잡도

전체 배열이 아닌 절반씩 탐색 범위를 줄여가기 때문에  
log(n)의 시간 복잡도로 탐색이 가능합니다.

> Time Complexity : O(log N)

<br>

## C++ 코드 예시

### 1. 반복문 풀이

```cpp
bool BS(int target) {
	int s = 0;
	int e = v.size() - 1;

	while (s <= e) {
		int mid = (s + e) / 2;
		if (v[mid] == target) return true;
		else if (v[mid] > target) e = mid - 1;
		else s = mid + 1;
	}

	return false;
}
```

<br>

### 2. 재귀 풀이

```cpp
bool BS(int s, int e, int tar) {
	if (s > e) return false;

	int mid = (s + e) / 2;
	if (v[mid] == tar) return true;
	else if (v[mid] > tar) return BS(s, mid-1, tar);
	else return BS(mid + 1, e, tar);
}
```

<br>

### 3. STL 이용 풀이 (binary_search 함수)

```cpp
#include <algorithm>

vector<int> v;
int target = 3;
bool check = binary_search(v.begin(), v.end(), target);
```

> binary_search(시작 인덱스, 마지막 인덱스, 찾고자 하는 값);  
> 값을 찾으면 true를, 찾지 못하면 false를 반환하는 함수입니다.

<br>

## 사용시 주의 사항

- 탐색 방향 반대로 설정하는 실수
  - v[mid] < target → 오른쪽 (mid + 1 ~ e)
  - v[mid] > target → 왼쪽 (s ~ mid - 1)

- while 조건을 s < e로 작성하는 실수
  - 반드시 s <= e (s와 e가 같을 때까지 탐색)

- 재귀 호출 시 return 누락

<br>

## 📌 핵심 정리

- 이진 탐색은 정렬된 배열에서만 사용 가능
- 탐색 범위를 절반씩 줄이며 탐색 → O(log N)
- mid 기준으로 탐색 방향 결정
