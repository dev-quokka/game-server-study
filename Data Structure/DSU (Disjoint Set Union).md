# DSU (Disjoint Set Union)

Disjoint Set (서로소 집합)은  
서로 겹치지 않는 여러 집합을 관리하면서  
집합을 합치거나(Union), 특정 원소가 어떤 집합에 속하는지(Find)  
빠르게 판단하기 위한 자료구조입니다.

Union-Find라고도 불리며, DSU와 동일한 의미입니다.

> 서로소: 공통 원소가 없는 집합

<br>

## DSU 주요 연산

- ### Find (찾기)

특정 원소가 속한 집합의 **대표 원소(루트 노드)**를 찾는 연산입니다.  
부모를 따라 올라가며 루트를 찾습니다.

<br>

<img src="../Data Structure/images/FIND1.png" width="592" height="236"/>

> 초기 상태에서는 자기 자신이 루트입니다.

<br>

<img src="../Data Structure/images/find2.png" width="600" height="245"/>

> 이미 합쳐진 경우, 최상위 부모(루트)를 반환합니다.

<br>

<br>

=> 위 방식은 트리의 높이가 커지면 Find 비용이 증가합니다.  
이를 해결하기 위해 **경로 압축**을 사용합니다.

<br>

<br>

- ### 경로 압축(path compression)

Find 연산 수행 시 탐색 경로에 있는 모든 노드를 **루트에 직접 연결**하는 최적화 기법입니다.  
이후 해당 노드들은 탐색 비용 없이 O(1)만에 find를 수행할 수 있습니다.

<br>

<img src="../Data Structure/images/경로압축.png" width="780" height="528"/>

> 최하위 노드부터 최상위 노드인 0에 직접 연결합니다.

<br>

#### 효과

- 트리 높이가 급격히 감소
- 이후 Find 연산이 거의 O(1)에 가까워짐

<br>

- ### Union (합치기)

두 개의 집합을 하나의 집합으로 합치는 연산입니다.  
즉, 한 트리의 루트를 다른 트리의 루트에 연결하여 두 트리를 하나로 결합합니다.

<br>

<img src="../Data Structure/images/유니온.png" width="780" height="524"/>

1. 두 원소의 루트를 각각 찾음 (Find)
2. 서로 다른 루트라면 한쪽을 다른 쪽에 연결

<br>

<br>

=> 위 방식은 트리가 한쪽으로 치우쳐 **높이가 증가**할 수 있습니다.  
이를 해결하기 위해 **Union by Rank** 사용

<br>

<br>

- ### Union by Rank

이 방법은 항상 작은 트리를 큰 트리 루트에 붙이는 방법입니다.  
여기서 작은 트리는 상대적으로 높이가 낮거나 사이즈가 작은 트리를 의미합니다.  
rank라는 배열을 사용하여 트리의 높이 or 사이즈를 저장합니다.

<br>

<img src="../Data Structure/images/유니온바이랭크.png" width="780" height="556"/>

- rank가 작은 트리를 큰 트리에 붙임
- rank가 같으면 한쪽으로 붙이고 rank 증가

<br>

#### 효과

- 트리 높이 증가 방지
- Find 성능 유지

<br>

## 예제 코드

```cpp
vector<int> parent, rankArr;

int Find(int x) {
    if (parent[x] == x)
        return x;
    return parent[x] = Find(parent[x]); // 경로 압축
}

void Union(int a, int b) {
    int rootA = Find(a);
    int rootB = Find(b);

    if (rootA == rootB) return;

    if (rankArr[rootA] < rankArr[rootB]) {
        parent[rootA] = rootB;
    }
    else if (rankArr[rootA] > rankArr[rootB]) {
        parent[rootB] = rootA;
    }
    else {
        parent[rootB] = rootA;
        rankArr[rootA]++;
    }
}
```

#### 동작 흐름 요약

1. 초기화  
   → 모든 노드는 자기 자신을 부모로 가짐

2. Find  
   → 루트 찾기 + 경로 압축

3. Union  
   → 두 집합 연결 + rank 기준 최적화

<br>

## 활용

- 그래프 사이클 판별
  - 간선을 추가할 때
  - 두 노드의 루트가 같다면 → 사이클 발생

- 최소 신장 트리 (Kruskal)
  - 간선을 가중치 기준으로 정렬
  - 사이클이 발생하지 않을 때만 Union

- 동적 연결 요소 관리
  - 네트워크 연결 여부 확인

<br>

## 📌 핵심 정리

- DSU는 서로소 집합을 관리하는 자료구조
- Find로 집합의 대표 원소(루트)를 찾을 수 있음
- Union으로 두 집합을 하나로 합칠 수 있음
- 경로 압축으로 Find 성능을 최적화할 수 있음
- Union by Rank로 트리 높이 증가를 줄일 수 있음
- 최적화 적용 시 거의 O(1)에 가까운 속도로 동작함

<br>

> DSU는 집합의 연결 여부를 빠르게 확인하고 병합해야 하는 문제에서 매우 효율적인 자료구조입니다.
