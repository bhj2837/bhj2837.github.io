---
title: "Decrease-and-Conquer (3): Variable-Size-Decrease"
date: 2026-02-23 19:00:00 +0900
categories: [Problem Solving Strategies, Decrease and Conquer]
tags: [algorithm, decrease-and-conquer, quickselect, interpolation-search, binary-search-tree]
math: true
---

## Variable-Size-Decrease란?

| 변형 | 감소량 | 예시 |
|---|---|---|
| Decrease-by-Constant | 매 단계 일정량 감소 (e.g. 1) | 삽입 정렬, 위상 정렬 |
| Decrease-by-Constant-Factor | 매 단계 일정 비율 감소 (e.g. 1/2) | 이진 탐색, 러시아 농부 |
| **Variable-Size-Decrease** | **매 단계 감소량이 다름** | 유클리드 GCD, Quickselect, 보간 탐색, BST |

Variable-Size-Decrease에서는 복잡도 분석이 단순한 점화식으로 표현되지 않는 경우가 많으며, 평균 복잡도 분석이 중요하다.

이 포스트에서 다루는 알고리즘은 다음과 같다.

| 알고리즘 | 문제 | 평균 복잡도 | 최악 복잡도 |
|---|---|---|---|
| Quickselect | $k$번째 최솟값 찾기 | $\Theta(n)$ | $\Theta(n^2)$ |
| Interpolation Search | 정렬 배열 탐색 | $\Theta(\log\log n)$ | $\Theta(n)$ |
| BST 탐색/삽입 | BST 연산 | $\Theta(\log n)$ | $\Theta(n)$ |

---

## 1. 선택 문제 (Selection Problem)와 Quickselect

### 문제 정의

**선택 문제**란 $n$개의 수 중에서 **$k$번째로 작은 원소($k$번째 순서 통계량)**를 찾는 문제다.

- $k = 1$: 최솟값 → 단순히 한 번 스캔으로 $\Theta(n)$
- $k = n$: 최댓값 → 마찬가지로 $\Theta(n)$
- $k = \lceil n/2 \rceil$: **중앙값(median)**

가장 단순한 접근법은 배열을 정렬한 뒤 $k$번째 원소를 반환하는 것이다. 병합 정렬 기준 $\Theta(n \log n)$이지만, 정렬은 전체 순서를 확정하는 반면 선택 문제는 $k$번째 원소만 찾으면 된다. 더 효율적인 방법이 있다.

### 핵심 아이디어: 분할(Partitioning)

배열의 첫 번째 원소를 **피벗(pivot) $p$**으로 설정하고, 배열을 다음 세 부분으로 재배치한다.

```
[ p보다 작은 원소들 | p | p보다 크거나 같은 원소들 ]
      < p            p         ≥ p
```

분할 후 피벗 $p$의 최종 위치를 $s$라 할 때(0-indexed), $p$는 전체 배열에서 정확히 $(s+1)$번째로 작은 원소다.

| 조건 | 다음 행동 |
|---|---|
| $s = k - 1$ | $p$가 $k$번째 최솟값 → **탐색 성공** |
| $s > k - 1$ | $k$번째 최솟값은 **왼쪽 부분**에 있음 |
| $s < k - 1$ | $k$번째 최솟값은 **오른쪽 부분**에 있고, 오른쪽에서 $(k - s)$번째 최솟값을 찾아야 함 |

이처럼 매 단계마다 탐색 범위가 피벗 위치에 따라 가변적으로 줄어들기 때문에 Variable-Size-Decrease에 해당한다.

### Lomuto 분할 (Lomuto Partitioning)

Quickselect에서 주로 사용하는 분할 방법이다. 포인터 $s$와 $i$를 활용하여 배열을 세 구간으로 관리한다.

```
[ l ] [  < p  ] [  ≥ p  ] [ ? ]
  p     l ~ s    s+1 ~ i-1   i ~ r
```

- $s$: "p보다 작은 원소" 구간의 오른쪽 경계
- $i$: 현재 검사 중인 원소

분할이 끝나면 피벗 $A[l]$을 $A[s]$와 교환하여 최종 배치를 완성한다.

```
[  < p  ] [ p ] [  ≥ p  ]
  l ~ s-1   s    s+1 ~ r
```

```cpp
// Lomuto 분할
// 입력: 배열 A의 부분 A[l..r]
// 출력: 피벗의 최종 위치 s
int lomutoPartition(std::vector<int>& A, int l, int r) {
    int p = A[l]; // 첫 번째 원소를 피벗으로 설정
    int s = l;
    for (int i = l + 1; i <= r; i++) {
        if (A[i] < p) {
            s++;
            std::swap(A[s], A[i]);
        }
    }
    std::swap(A[l], A[s]); // 피벗을 최종 위치로 이동
    return s;
}
```

### Quickselect 알고리즘

```cpp
// Quickselect: A[l..r]에서 k번째 최솟값 반환 (1-indexed)
int quickselect(std::vector<int>& A, int l, int r, int k) {
    int s = lomutoPartition(A, l, r);
    if (s == l + k - 1)        // 피벗이 정확히 k번째 위치
        return A[s];
    else if (s > l + k - 1)    // k번째는 왼쪽 부분에 있음
        return quickselect(A, l, s - 1, k);
    else                        // k번째는 오른쪽 부분에 있음
        return quickselect(A, s + 1, r, l + k - 1 - s);
}
```

### 동작 예시

배열 `[4, 1, 10, 8, 7, 12, 9, 2, 15]`에서 **5번째 최솟값(중앙값)** 찾기. ($k = 5$)

**1차 분할** (피벗 = 4, 전체 배열 인덱스 0~8):

| 단계 | 배열 상태 |
|---|---|
| 초기 | `4  1 10  8  7 12  9  2 15` (s=0, i=1) |
| i=1: 1<4 | `4  1 10  8  7 12  9  2 15` (s=1, swap A[1]↔A[1]) |
| i=2~6: 모두 ≥4 | s 변화 없음 |
| i=7: 2<4 | `4  1  2  8  7 12  9 10 15` (s=2, swap A[2]↔A[7]) |
| i=8: 15≥4 | s 변화 없음 |
| 최종 swap | `2  1  4  8  7 12  9 10 15` (A[0]↔A[2]) |

피벗 4의 위치 $s = 2$. $s < k-1 = 4$이므로 오른쪽 부분 `[8, 7, 12, 9, 10, 15]` (인덱스 3~8)에서 $(5 - 2 - 1) = 2$번째 최솟값을 찾는다.

**2차 분할** (피벗 = 8, 인덱스 3~8):

| 단계 | 배열 상태 (인덱스 3~8) |
|---|---|
| 초기 | `8  7 12  9 10 15` |
| i=4: 7<8 | swap → `8  7 12  9 10 15` (s=4) |
| i=5~8: 모두 ≥8 | s 변화 없음 |
| 최종 swap | `7  8 12  9 10 15` (A[3]↔A[4]) |

피벗 8의 위치 $s = 4$. $s = l + k - 1 = 3 + 2 - 1 = 4$ → **탐색 성공!** 중앙값은 **8**이다.

### 복잡도 분석

분할 시 $n$개 원소에 대해 항상 $n-1$번의 비교가 필요하다.

| 케이스 | 발생 조건 | 복잡도 |
|---|---|---|
| 최선 | 첫 분할에서 바로 $s = k-1$ | $C_{best}(n) = n - 1 \in \Theta(n)$ |
| 최악 | 매번 가장 불균형한 분할 (e.g. 항상 피벗이 최대값) | $C_{worst}(n) = (n-1)+(n-2)+\cdots+1 = \dfrac{n(n-1)}{2} \in \Theta(n^2)$ |
| 평균 | 피벗 위치가 균등 분포 | $C_{avg}(n) \in \Theta(n)$ |

평균 복잡도가 $\Theta(n)$이라는 점이 핵심이다. 정렬($\Theta(n \log n)$)보다 빠르게 $k$번째 원소를 찾을 수 있다. 알고리즘의 실용적 가치는 바로 이 평균 선형 복잡도에 있다.

---

## 2. 보간 탐색 (Interpolation Search)

### 개념

이진 탐색은 정렬된 배열에서 탐색 키를 항상 **중간 위치**에서 찾는다. 하지만 우리가 사전에서 단어를 찾을 때는 다르다. "Smith"를 찾을 때는 사전 뒷부분을, "Brown"을 찾을 때는 앞부분을 펼친다. 이처럼 **탐색 키의 값을 활용**해 비교 위치를 동적으로 결정하는 것이 보간 탐색이다.

| 탐색 방법 | 비교 위치 결정 방식 |
|---|---|
| 이진 탐색 | 항상 현재 구간의 **중간** |
| 보간 탐색 | 탐색 키의 **값에 비례**하는 위치 |

### 보간 위치 계산

탐색 구간이 $A[l..r]$이고 탐색 키가 $v$일 때, 보간으로 추정한 탐색 위치 $x$는 다음과 같다.

$$x = l + \left\lfloor \frac{(v - A[l])(r - l)}{A[r] - A[l]} \right\rfloor$$

이 식은 $A[l]$과 $A[r]$을 잇는 직선 위에서 값 $v$에 대응하는 인덱스를 선형 보간(linear interpolation)으로 구한 것이다. 배열의 값이 균등 분포(uniform distribution)에 가까울수록 정확한 위치를 추정한다.

### 이진 탐색 vs 보간 탐색

| 항목 | 이진 탐색 | 보간 탐색 |
|---|---|---|
| 비교 위치 | 항상 중간 | 키 값에 따라 가변 |
| 감소 방식 | Decrease-by-Half | Variable-Size-Decrease |
| 평균 복잡도 | $\Theta(\log n)$ | $\Theta(\log \log n)$ (균등 분포 가정) |
| 최악 복잡도 | $\Theta(\log n)$ | $\Theta(n)$ |
| 적합한 상황 | 일반적인 정렬 배열 | 값이 균등하게 분포된 대용량 배열 |

평균 복잡도 $\Theta(\log \log n)$은 이진 탐색보다 훨씬 빠르다. $n = 2^{16} = 65536$이면 이진 탐색은 약 16번의 비교, 보간 탐색은 약 4번의 비교만 필요하다.

그러나 값이 극도로 편향된 경우(e.g. 기하급수적으로 증가하는 수열) 최악의 경우 $\Theta(n)$이 될 수 있다. 이것이 보간 탐색의 단점이다.

### C++ 구현

```cpp
// 보간 탐색
// 입력: 오름차순 정렬된 배열 A, 크기 n, 탐색 키 v
// 출력: v의 인덱스, 없으면 -1
int interpolationSearch(int A[], int n, int v) {
    int l = 0, r = n - 1;
    while (l <= r && v >= A[l] && v <= A[r]) {
        if (l == r) {
            return (A[l] == v) ? l : -1;
        }
        // 보간 위치 계산
        int x = l + (long long)(v - A[l]) * (r - l) / (A[r] - A[l]);
        if (A[x] == v)       return x;
        else if (A[x] < v)   l = x + 1;
        else                  r = x - 1;
    }
    return -1;
}
```

`(long long)` 캐스팅은 정수 오버플로우 방지를 위한 것이다.

---

## 3. 이진 탐색 트리 (Binary Search Tree)

이진 탐색 트리(BST)에서의 탐색, 삽입, 최솟값/최댓값 찾기는 모두 Variable-Size-Decrease의 대표 예시다.

루트 키 $k$와 탐색 키를 비교하여:
- 같으면 탐색 성공
- 작으면 왼쪽 서브트리만 재귀 처리
- 크면 오른쪽 서브트리만 재귀 처리

각 단계에서 탐색 범위가 줄어드는 양은 해당 서브트리의 크기에 따라 달라지므로 Variable-Size-Decrease다. BST가 균형 잡혀 있으면 평균 $\Theta(\log n)$, 트리가 완전히 치우치면(예: 오름차순으로 삽입) 최악 $\Theta(n)$이 된다.

BST의 자료구조와 연산에 대한 상세한 내용은 **Data Structures** 파트에서 별도로 다룰 예정이다. 여기서는 BST 탐색이 Variable-Size-Decrease 패러다임에 속한다는 점만 확인해두자.

---

## Variable-Size-Decrease 알고리즘 비교

| 알고리즘 | 감소 결정 요인 | 평균 복잡도 | 최악 복잡도 |
|---|---|---|---|
| 유클리드 GCD | $\gcd(m, n) = \gcd(n, m \bmod n)$ | $\Theta(\log n)$ | $\Theta(\log n)$ |
| Quickselect | 피벗의 위치 $s$ | $\Theta(n)$ | $\Theta(n^2)$ |
| Interpolation Search | 탐색 키 $v$의 값 | $\Theta(\log\log n)$ | $\Theta(n)$ |
| BST 탐색 | 각 서브트리의 크기 | $\Theta(\log n)$ | $\Theta(n)$ |

Variable-Size-Decrease 알고리즘의 공통적인 특징은 **평균 복잡도와 최악 복잡도 사이에 큰 차이**가 있다는 점이다. 따라서 최악의 경우보다 평균적인 성능이 실용적으로 더 중요하다.

---

