# Bellman-Ford Algorithm

## 벨만-포드 알고리즘이란?

벨만-포드(Bellman-Ford) 알고리즘은  
하나의 시작 정점에서 다른 모든 정점까지의 최단 거리를 구하는 알고리즘입니다.

다익스트라와 유사하지만  
**음수 가중치 간선이 있어도 정상적으로 동작**한다는 점이 가장 큰 특징입니다.

또한, **그래프 내 음수 사이클(Negative Cycle) 존재 여부까지 판별 가능**합니다.

<br>

## 왜 필요한가?

최단 경로 문제에서는 보통 다익스트라를 많이 사용하지만,  
다익스트라는 "현재까지의 최단 거리는 확정이다" (Greedy)를 기반으로 동작하기 때문에  
음수 간선이 있는 경우 올바르게 동작하지 않을 수 있습니다.

반면 벨만-포드는 **모든 간선을 반복적으로 확인**하기 때문에  
뒤늦게 더 짧은 경로가 발견되어도 반영할 수 있습니다.

<br>

## 구현 핵심 아이디어

### 1. Relaxation (간선 완화)

벨만-포드는 **모든 간선을 반복적으로 확인하며 거리 값을 갱신(Relaxation)** 합니다.

```cpp
if (dist[from] != INF && dist[to] > dist[from] + cost) {
    dist[to] = dist[from] + cost;
}
```

> 더 짧은 경로가 발견되면 기존 값을 덮어쓰며 최단 거리를 갱신

<br>

### 2. V-1번 반복

정점이 V개일 때,
사이클이 없는 최단 경로는 최대 V-1개의 간선을 가집니다.

따라서 모든 간선을 V-1번 반복하면  
최단 거리 정보가 그래프 전체로 전파됩니다.

```
1 → 2 → 3 → 4
```

1차 반복: 1 → 2 갱신  
2차 반복: 2 → 3 갱신  
3차 반복: 3 → 4 갱신

> 반복할 수록 단계적으로 퍼짐

<br>

### 3. 음수 사이클 탐지

V-1번 반복 이후에는
최단 거리가 확정되어야 정상입니다.

하지만 추가 반복에서 값이 더 줄어든다면  
거리가 계속 감소하는 경로가 존재한다는 의미입니다. (음수 사이클 존재)

```cpp
if (dist[from] != INF && dist[to] > dist[from] + cost) {
    // 음수 사이클 존재
}
```

> 시작점에서 도달 가능한 사이클만 의미 있음

<br>

<br>

- 위 과정을 활용하여 벨만포드는 아래처럼 동작합니다.

```
1. 시작 정점 거리 = 0
2. 나머지 정점 = INF
3. 모든 간선을 V-1번 반복
4. 한 번 더 검사해서 갱신되면 → 음수 사이클 존재
```

<br>

#### 시간복잡도

> O(VE)  
> 모든 간선(E)을 V-1번 확인

<br>

## 다익스트라와 차이

| 구분        | 다익스트라          | 벨만-포드                         |
| ----------- | ------------------- | --------------------------------- |
| 동작 방식   | Greedy              | 반복 완화                         |
| 음수 간선   | 불가능              | 가능                              |
| 사이클 탐지 | 불가능              | 가능                              |
| 시간복잡도  | O(E log V)          | O(VE)                             |
| 사용 목적   | 빠른 최단 거리 탐색 | 음수 간선 탐색 / 음수 사이클 확인 |

<br>

## 구현 시 주의사항

- dist[u] != INF 체크 필수 (오버플로우 방지)
- 거리 배열은 long long 사용 권장
- 음수 사이클은 시작점에서 도달 가능한 경우만 의미 있음
- 간선 리스트(Edge List) 형태로 구현하는 것이 일반적  
  → 모든 간선을 반복해야 하기 때문

<br>

## 구현 코드

```cpp
#include <iostream>
#include <vector>

using namespace std;

struct Edge {
    int from, to, cost;
};

const long long INF = 1e18;

int main() {
    int V, E, start;
    cin >> V >> E >> start;

    vector<Edge> edges;
    vector<long long> dist(V + 1, INF);

    for (int i = 0; i < E; i++) {
        int a, b, c; cin >> a >> b >> c;
        edges.push_back({a, b, c});
    }

    dist[start] = 0;

    // V-1번 반복
    for (int i = 1; i <= V - 1; i++) {
        bool updated = false;

        for (const auto& [from, to, cost] : edges) {
            if (dist[from] != INF && dist[to] > dist[from] + cost) {
                dist[to] = dist[from] + cost;
                updated = true;
            }
        }

        // 더 이상 갱신이 없으면 조기 종료
        if (!updated) break;
    }

    // 음수 사이클 확인
    bool negativeCycle = false;

    for (const auto& [from, to, cost] : edges) {
        if (dist[from] != INF && dist[to] > dist[from] + cost) { // 음수 사이클 확인
            negativeCycle = true;
            break;
        }
    }

    if (negativeCycle) cout << "Negative Cycle Exists" << '\n';
    else {
        for (int i = 1; i <= V; i++) {
            if (dist[i] == INF) cout << "INF" << '\n';
            else cout << dist[i] << '\n';
        }
    }

    return 0;
}
```

<br>

## 언제 사용하면 좋을까?

- 음수 간선이 존재할 수 있을 때
- 음수 사이클 존재 여부를 판별해야 할 때
- 정확성이 중요한 경우

<br>

음수 간선이 없고 빠른 속도가 중요하다면

→ 다익스트라 사용

<br>

## 📌 핵심 정리

- 벨만-포드는 음수 간선이 있어도 최단 거리 계산 가능
- 모든 간선을 반복 확인하며 거리 갱신
- 정점이 V개라면 최대 V-1번 완화
- 한 번 더 갱신되면 음수 사이클 존재
- 시간복잡도는 O(VE) 로 다익스트라보다 느림

<br>

> 벨만-포드는 “음수 간선까지 처리 가능한 최단 경로 알고리즘” 이며  
> 음수 사이클 탐지까지 가능하다는 점이 핵심입니다.
