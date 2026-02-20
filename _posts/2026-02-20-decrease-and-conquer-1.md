---
title: "Decrease-and-Conquer (1): 개념과 Decrease-by-Constant 알고리즘"
date: 2026-02-20 18:00:00 +0900
categories: [Problem Solving Strategies, Decrease and Conquer]
tags: [algorithm, decrease-and-conquer, insertion-sort, topological-sort, dfs]
math: true
---

## Decrease-and-Conquer란?

Decrease-and-Conquer는 주어진 문제의 인스턴스와 그보다 작은 인스턴스의 해 사이의 관계를 이용하는 알고리즘 설계 기법이다.

핵심 아이디어는 단순하다. 크기 $n$인 문제를 풀기 위해 크기 $n-1$ (혹은 $n/2$, 혹은 그보다 작은) 문제 하나로 환원하고, 그 작은 문제의 해를 이용해 원래 문제의 해를 구한다. 이 관계가 확립되면 두 가지 방식으로 구현할 수 있다.

**Top-down (재귀)**은 큰 문제에서 출발해 작은 문제로 반복적으로 내려가며 재귀 호출로 구현한다. 최종적으로는 비재귀적으로도 구현 가능한 경우가 많다.

**Bottom-up (반복)**은 가장 작은 인스턴스의 해에서 출발해 점진적으로 크기를 키워나간다. 이를 **incremental approach(점진적 접근법)**라고도 한다. 삽입 정렬이 대표적인 예시다.

---

## 세 가지 변형

Decrease-and-Conquer는 문제 크기를 줄이는 방식에 따라 세 가지로 분류된다.

| 변형 | 설명 | 예시 |
|------|------|------|
| Decrease by a constant | 매 단계 크기를 **고정값(보통 1)**만큼 줄임 | 삽입 정렬, 위상 정렬 |
| Decrease by a constant factor | 매 단계 크기를 **비율(보통 1/2)**로 줄임 | 이진 탐색, 러시아 농부 곱셈법 |
| Variable-size decrease | 단계마다 줄이는 크기가 **가변적** | 유클리드 호제법 |

이번 포스트에서는 첫 번째 변형인 **Decrease by a Constant**를 중심으로 다룬다.

---

## Decrease by a Constant

매 단계 인스턴스의 크기를 동일한 상수만큼 줄이는 방식이다. 상수는 대부분 1이다.

### 거듭제곱 문제로 이해하기

$a^n$을 계산하는 문제를 생각해보자. 크기 $n$짜리 인스턴스와 크기 $n-1$짜리 인스턴스 사이에는 다음의 명확한 관계가 성립한다.

$$a^n = a^{n-1} \cdot a$$

**Top-down**으로 구현하면 재귀 점화식이 된다.

$$f(n) = \begin{cases} f(n-1) \cdot a & \text{if } n > 0 \\ 1 & \text{if } n = 0 \end{cases}$$

**Bottom-up**으로 구현하면 1에 $a$를 $n$번 곱하는 것과 동일하다. 이는 이전에 다룬 브루트 포스 알고리즘과 결과적으로 같다.

이 예시는 단순하지만, Decrease-and-Conquer의 핵심 구조인 "큰 문제를 작은 문제 하나로 환원"을 잘 보여준다.

---

## 삽입 정렬 (Insertion Sort)

### Decrease-and-Conquer 관점에서의 해석

삽입 정렬은 이전 포스트에서 점진적 접근법의 예시로 다룬 바 있다. 이번에는 Decrease-and-Conquer의 관점에서 다시 해석해보자.

배열 `A[0..n-1]`을 정렬하는 문제는 다음과 같이 환원된다.

> `A[0..n-2]`를 재귀적으로 정렬한 뒤, `A[n-1]`을 정렬된 `A[0..n-2]` 내의 올바른 위치에 삽입한다.

이것이 Decrease-by-one-and-conquer의 전형적인 구조다. 크기 $n$ 문제를 크기 $n-1$ 문제 하나로 환원하고, 그 결과를 이용해 원래 문제를 해결한다.

재귀적 아이디어를 갖고 있지만, 실제 구현은 보통 **Bottom-up(반복)** 방식으로 이루어진다.

### 동작 예시

`6, 4, 1, 8, 5`를 정렬하는 과정을 단계별로 살펴보자. `|` 기호는 정렬된 부분과 미정렬 부분의 경계를 나타낸다.

```
6 | 4  1  8  5     → A[1]=4를 정렬된 부분에 삽입: 6 앞에 위치
4  6 | 1  8  5     → A[2]=1을 정렬된 부분에 삽입: 4 앞에 위치
1  4  6 | 8  5     → A[3]=8을 정렬된 부분에 삽입: 6 뒤에 위치
1  4  6  8 | 5     → A[4]=5를 정렬된 부분에 삽입: 4와 6 사이에 위치
1  4  5  6  8      → 정렬 완료
```

각 단계에서 `A[i]`를 `v`에 저장하고, `v`보다 큰 원소들을 오른쪽으로 한 칸씩 밀어낸 뒤 `v`를 빈 자리에 삽입한다.

### 루프 불변식 (Loop Invariant)

삽입 정렬의 정확성은 다음 루프 불변식으로 보장된다.

> `i`번째 반복이 시작될 때, `A[0..i-1]`은 원래 `A[0..i-1]`에 있던 원소들이 오름차순으로 정렬된 상태다.

- **초기화**: `i=1`일 때 `A[0..0]`은 원소가 하나뿐이므로 자명하게 정렬되어 있다.
- **유지**: 각 반복에서 `A[i]`를 `A[0..i-1]`의 올바른 위치에 삽입하므로 `A[0..i]`가 정렬된 상태가 된다.
- **종료**: `i=n`이 되면 `A[0..n-1]` 전체가 정렬된다.

### C++ 코드

```cpp
#include <iostream>
#include <vector>
using namespace std;

void insertionSort(vector<int>& A) {
    int n = A.size();
    for (int i = 1; i < n; i++) {
        int v = A[i];   // 삽입할 원소 저장
        int j = i - 1;
        // v보다 큰 원소들을 오른쪽으로 한 칸씩 이동
        while (j >= 0 && A[j] > v) {
            A[j + 1] = A[j];
            j--;
        }
        A[j + 1] = v;   // 올바른 위치에 삽입
    }
}

int main() {
    vector<int> A = {6, 4, 1, 8, 5};
    insertionSort(A);
    for (int x : A) cout << x << " ";
    // 출력: 1 4 5 6 8
    return 0;
}
```

**코드 설명:**
- 외부 루프: `i`는 삽입할 원소의 인덱스. 1부터 시작 (A[0]은 이미 정렬된 상태).
- `v = A[i]`: 현재 삽입할 원소를 임시 저장.
- 내부 `while` 루프: `v`보다 큰 원소를 오른쪽으로 밀면서 삽입 위치를 탐색.
- `A[j+1] = v`: 빈 자리에 `v`를 삽입.

### 복잡도 분석

기본 연산은 **키 비교 `A[j] > v`**다.

**최악의 경우 (Worst Case)**: 입력 배열이 역순으로 정렬된 경우, `i`번째 단계에서 `i`번의 비교가 발생한다.

$$C_{worst}(n) = \sum_{i=1}^{n-1} \sum_{j=0}^{i-1} 1 = \sum_{i=1}^{n-1} i = \frac{(n-1)n}{2} \in \Theta(n^2)$$

**최선의 경우 (Best Case)**: 입력 배열이 이미 오름차순으로 정렬된 경우, 각 단계에서 비교가 1번만 발생한다.

$$C_{best}(n) = \sum_{i=1}^{n-1} 1 = n - 1 \in \Theta(n)$$

**평균의 경우 (Average Case)**: 임의 순서의 배열에서 각 단계에 평균적으로 절반 정도의 비교가 발생하므로, 최악 경우의 절반 수준이다.

$$C_{avg}(n) \approx \frac{n^2}{4} \in \Theta(n^2)$$

---

## 위상 정렬 (Topological Sorting)

### DAG (Directed Acyclic Graph)

위상 정렬을 이해하려면 먼저 **DAG**를 알아야 한다. DAG는 방향 비순환 그래프(Directed Acyclic Graph)로, 방향 간선이 있으면서 사이클이 존재하지 않는 그래프다.

DAG는 선행 조건이 있는 작업을 모델링하는 데 자주 사용된다. 예를 들어 수강 선수과목 구조를 생각해보자. C1과 C2를 수강해야 C3를 수강할 수 있고, C3와 C4를 수강해야 C5를 수강할 수 있다고 하면 다음과 같은 DAG로 표현된다.

```
C1 → C3 ← C2
     ↓
C4 → C5
```

### 위상 정렬의 정의

위상 정렬은 DAG의 모든 정점을 선형으로 나열하되, 모든 간선 (u, v)에 대해 u가 v보다 앞에 오도록 나열하는 것이다. 위상 정렬이 가능하려면 반드시 DAG여야 한다 — 사이클이 존재하면 선형 순서를 만들 수 없다.

위 예시의 경우 `C1, C2, C3, C4, C5` 또는 `C2, C1, C4, C3, C5` 등 여러 개의 올바른 답이 존재할 수 있다.

### 알고리즘 1: DFS 기반

DFS(깊이 우선 탐색)를 수행하면서 **dead-end(더 이상 방문할 이웃이 없는 정점)**가 되는 순서를 기록하고, 그 순서를 역순으로 나열하면 위상 정렬이 완성된다.

**동작 예시** (C1~C5 그래프):

DFS를 수행하면 스택에서 pop되는 순서가 다음과 같다고 하자.

```
pop 순서: C5, C4, C3, C1, C2
역순:     C2, C1, C3, C4, C5
```

즉 `C2 → C1 → C3 → C4 → C5`가 위상 정렬 결과가 된다.

**C++ 코드:**

```cpp
#include <iostream>
#include <vector>
#include <stack>
#include <algorithm>
using namespace std;

int n; // 정점 수
vector<int> adj[100]; // 인접 리스트
bool visited[100];
stack<int> result;

void dfs(int v) {
    visited[v] = true;
    for (int w : adj[v]) {
        if (!visited[w])
            dfs(w);
    }
    result.push(v); // dead-end가 될 때 스택에 push
}

void topologicalSortDFS() {
    fill(visited, visited + n, false);
    for (int v = 0; v < n; v++) {
        if (!visited[v])
            dfs(v);
    }
    cout << "위상 정렬 결과: ";
    while (!result.empty()) {
        cout << result.top() << " ";
        result.pop();
    }
    cout << endl;
}
```

DFS 기반 위상 정렬의 시간 복잡도는 $O(V + E)$다. 모든 정점과 간선을 한 번씩 방문하기 때문이다.

### 알고리즘 2: Source-Removal 알고리즘

Source-Removal 알고리즘은 Decrease-by-one-and-conquer의 구조를 더 직관적으로 드러낸다.

> **소스(source)**: 진입 차수(incoming edge의 수)가 0인 정점

알고리즘은 다음을 반복한다.

1. 현재 그래프에서 소스를 하나 찾는다.
2. 소스를 출력하고, 소스와 그로부터 나가는 모든 간선을 그래프에서 제거한다.
3. 그래프가 빌 때까지 반복한다.

소스를 제거할 때마다 그래프의 정점 수가 1 감소하므로, 이것이 바로 **Decrease-by-one**이다.

**동작 예시** (C1~C5 그래프):

```
초기 상태: C1, C2, C3, C4, C5  (소스: C1, C2)
C1 제거  → C2, C3, C4, C5       (소스: C2)
C2 제거  → C3, C4, C5           (소스: C3, C4)
C3 제거  → C4, C5               (소스: C4)
C4 제거  → C5                   (소스: C5)
C5 제거  →  (완료)
결과: C1, C2, C3, C4, C5
```

소스가 여러 개인 경우 어떤 것을 먼저 선택하느냐에 따라 다른 결과가 나올 수 있으므로, 위상 정렬의 해는 유일하지 않다.

**C++ 코드:**

```cpp
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

void topologicalSortSourceRemoval(int n, vector<vector<int>>& adj) {
    vector<int> inDegree(n, 0);

    // 각 정점의 진입 차수 계산
    for (int v = 0; v < n; v++)
        for (int w : adj[v])
            inDegree[w]++;

    // 진입 차수가 0인 소스를 큐에 삽입
    queue<int> sources;
    for (int v = 0; v < n; v++)
        if (inDegree[v] == 0)
            sources.push(v);

    cout << "위상 정렬 결과: ";
    while (!sources.empty()) {
        int v = sources.front();
        sources.pop();
        cout << v << " ";

        // v에서 나가는 간선 제거 → 인접 정점의 진입 차수 감소
        for (int w : adj[v]) {
            inDegree[w]--;
            if (inDegree[w] == 0)
                sources.push(w); // 새로운 소스 발생
        }
    }
    cout << endl;
}
```

Source-Removal 알고리즘의 시간 복잡도도 $O(V + E)$다.

### 두 알고리즘 비교

| | DFS 기반 | Source-Removal |
|---|---|---|
| 구현 방식 | 재귀 DFS + 스택 | 진입 차수 배열 + 큐 |
| 시간 복잡도 | $O(V + E)$ | $O(V + E)$ |
| Decrease-and-Conquer 연결 | Top-down 재귀 | Decrease-by-one 직관적 표현 |
| 사이클 감지 | 가능 (back edge) | 가능 (큐가 비어도 출력 수 < V) |

---

## Variable-size Decrease 맛보기

세 번째 변형인 Variable-size decrease는 매 단계 줄어드는 크기가 일정하지 않다.

대표적인 예가 **유클리드 호제법**이다.

$$\gcd(m, n) = \gcd(n, m \bmod n)$$

$m \bmod n$의 값은 $m$과 $n$에 따라 달라지므로, 줄어드는 크기가 매 단계 가변적이다. constant도 아니고, constant factor도 아닌 것이 Variable-size decrease의 핵심이다. 이 변형의 구체적인 알고리즘들은 이후 포스트에서 자세히 다룬다.
