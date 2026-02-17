---
title: "브루트 포스 (Brute Force) - 2"
date: 2025-12-03 18:00:00 +0900
categories: [Algorithm Basics, Brute Force]
tags: [algorithm, brute-force, search, string-matching, closest-pair, convex-hull]
math: true
---

## 순차 탐색(Sequential Search)

순차 탐색은 리스트에서 특정 값을 찾기 위해 처음부터 끝까지 순서대로 확인하는 가장 기본적인 탐색 알고리즘이다.

### 기본 알고리즘

리스트의 연속된 요소들을 주어진 탐색 키와 비교하여:
- **성공적 탐색(Successful search)**: 일치하는 값을 찾음
- **비성공적 탐색(Unsuccessful search)**: 리스트를 모두 확인했으나 찾지 못함

### C++ 구현 (기본)

```cpp
#include <vector>
using namespace std;

int sequentialSearch(const vector<int>& arr, int key) {
    for (int i = 0; i < arr.size(); i++) {
        if (arr[i] == key) {
            return i;  // 찾은 위치 반환
        }
    }
    return -1;  // 찾지 못함
}
```

### Sentinel을 이용한 최적화

**Sentinel 기법**: 탐색 키를 리스트의 끝에 추가하면 리스트 끝 검사를 제거할 수 있다.

```cpp
int sequentialSearchSentinel(vector<int>& arr, int key) {
    int n = arr.size();
    arr.push_back(key);  // Sentinel 추가
    
    int i = 0;
    while (arr[i] != key) {
        i++;
    }
    
    arr.pop_back();  // Sentinel 제거
    
    if (i < n) return i;  // 찾음
    return -1;  // 찾지 못함
}
```

**장점**: 매 반복마다 `i < n` 검사를 하지 않아도 된다.

### 정렬된 리스트에서의 최적화

리스트가 **정렬되어 있다면**, 탐색 키보다 큰 값을 만나는 순간 탐색을 중단할 수 있다.

```cpp
int sequentialSearchSorted(const vector<int>& arr, int key) {
    int i = 0;
    while (i < arr.size() && arr[i] < key) {
        i++;
    }
    
    if (i < arr.size() && arr[i] == key) {
        return i;  // 찾음
    }
    return -1;  // 찾지 못함
}
```

### 시간 복잡도

- **최선의 경우**: O(1) - 첫 번째 원소가 탐색 키
- **최악의 경우**: O(n) - 마지막 원소이거나 없는 경우
- **평균의 경우**: O(n/2) = O(n)

## 브루트 포스 문자열 매칭(String Matching)

문자열 매칭 문제는 텍스트(text)에서 패턴(pattern)을 찾는 문제다.

### 문제 정의

- **패턴(Pattern)**: 찾고자 하는 문자열 (길이 m)
- **텍스트(Text)**: 검색 대상 문자열 (길이 n)
- **목표**: 텍스트에서 패턴이 처음 나타나는 위치 찾기

### 브루트 포스 알고리즘

**Step 1**: 패턴을 텍스트의 시작 부분에 정렬한다

**Step 2**: 왼쪽에서 오른쪽으로 이동하며 패턴의 각 문자를 텍스트의 대응 문자와 비교한다
- 모든 문자가 일치하면 → 성공
- 불일치가 발견되면 → Step 3으로

**Step 3**: 패턴을 한 칸 오른쪽으로 이동하고 Step 2를 반복한다

### 알고리즘 의사코드

```
ALGORITHM BruteForceStringMatch(T[0..n-1], P[0..m-1])
    // 브루트 포스 문자열 매칭 구현
    // Input: 텍스트 T (n개 문자), 패턴 P (m개 문자)
    // Output: 매칭되는 부분 문자열의 시작 인덱스, 없으면 -1
    
    for i ← 0 to n-m do
        j ← 0
        while j < m and P[j] = T[i+j] do
            j ← j + 1
        if j = m return i
    return -1
```

### C++ 구현

```cpp
#include <string>
using namespace std;

int bruteForceStringMatch(const string& text, const string& pattern) {
    int n = text.length();
    int m = pattern.length();
    
    // 텍스트의 각 위치에서 시도
    for (int i = 0; i <= n - m; i++) {
        int j = 0;
        
        // 패턴의 모든 문자 비교
        while (j < m && pattern[j] == text[i + j]) {
            j++;
        }
        
        // 모든 문자가 일치하면 찾음
        if (j == m) {
            return i;
        }
    }
    
    return -1;  // 찾지 못함
}
```

### 동작 과정 예시

**텍스트**: `"NOBODY_NOTICED_HIM"`  
**패턴**: `"NOT"`

```
위치 0: NOB (X - 'B' != 'T')
        NOT
        
위치 1: OBO (X - 'O' != 'N')
         NOT
         
위치 2: BOD (X - 'B' != 'N')
          NOT
          
...

위치 7: NOT (O - 일치!)
               NOT
               
위치 반환: 7
```

### 시간 복잡도 분석

**최악의 경우**: 

모든 위치에서 거의 끝까지 비교해야 하는 경우

$$C(n,m) = m(n-m+1) = O(nm)$$

예: 텍스트 `"AAAAAAAAAB"`, 패턴 `"AAAB"`

**평균의 경우**:

자연어 텍스트에서는 대부분의 shift가 매우 적은 비교 후에 발생한다.
- 실제로는 거의 **선형 시간 O(n)**에 가깝다

**랜덤 텍스트**:
- 이론적으로 Θ(n)으로 증명됨

## 최근접 쌍 문제(Closest-Pair Problem)

평면 또는 고차원 공간에서 n개의 점 중 **가장 가까운 두 점**을 찾는 문제다.

### 문제의 중요성

계산 기하학(Computational Geometry)에서 가장 기본적인 문제 중 하나로:
- **클러스터 분석**: 가장 가까운 점들을 찾아 군집화
- **충돌 감지**: 컴퓨터 그래픽스, 게임
- **데이터 분석**: 유사한 데이터 포인트 찾기
- **통계적 이상치 탐지**

점들은 비행기, 우편함 같은 물리적 객체뿐만 아니라 데이터베이스 레코드, 통계 샘플, DNA 시퀀스 등을 나타낼 수 있다.

### 브루트 포스 알고리즘

**아이디어**: 모든 점 쌍의 거리를 계산하여 최솟값을 찾는다.

**알고리즘**:
1. 모든 점 쌍 $(p_i, p_j)$ (i < j)에 대해
2. 거리 $d(p_i, p_j)$를 계산
3. 최소 거리를 가진 쌍을 반환

### 의사코드

```
ALGORITHM BruteForceClosestPair(P)
    // 브루트 포스로 최근접 점 쌍 찾기
    // Input: n개 점의 리스트 P
    // Output: 최근접 점 쌍 사이의 거리
    
    d ← ∞
    for i ← 1 to n-1 do
        for j ← i+1 to n do
            d ← min(d, sqrt((x_i - x_j)² + (y_i - y_j)²))
    return d
```

### C++ 구현

```cpp
#include <vector>
#include <cmath>
#include <limits>
using namespace std;

struct Point {
    double x, y;
};

double distance(const Point& p1, const Point& p2) {
    double dx = p1.x - p2.x;
    double dy = p1.y - p2.y;
    return sqrt(dx * dx + dy * dy);
}

double bruteForceClosestPair(const vector<Point>& points) {
    int n = points.size();
    double minDist = numeric_limits<double>::infinity();
    
    // 모든 점 쌍의 거리 계산
    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            double d = distance(points[i], points[j]);
            minDist = min(minDist, d);
        }
    }
    
    return minDist;
}

// 최근접 쌍의 인덱스도 반환하는 버전
pair<int, int> bruteForceClosestPairIndices(const vector<Point>& points) {
    int n = points.size();
    double minDist = numeric_limits<double>::infinity();
    int idx1 = -1, idx2 = -1;
    
    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            double d = distance(points[i], points[j]);
            if (d < minDist) {
                minDist = d;
                idx1 = i;
                idx2 = j;
            }
        }
    }
    
    return {idx1, idx2};
}
```

### 시간 복잡도 분석

**기본 연산**: 거리 계산 (제곱근 계산)

점 쌍의 개수:

$$\binom{n}{2} = \frac{n(n-1)}{2}$$

**비교 횟수**:

$$C(n) = \sum_{i=1}^{n-1} \sum_{j=i+1}^{n} 2 = 2\sum_{i=1}^{n-1}(n-i)$$

$$= 2[(n-1) + (n-2) + \cdots + 1] = 2 \cdot \frac{(n-1)n}{2} = (n-1)n$$

**시간 복잡도**: **Θ(n²)**

### 최적화 팁

**제곱근 계산 생략**:

거리 비교만 하면 되므로 제곱근을 계산하지 않아도 된다:

```cpp
double distanceSquared(const Point& p1, const Point& p2) {
    double dx = p1.x - p2.x;
    double dy = p1.y - p2.y;
    return dx * dx + dy * dy;  // sqrt 생략
}
```

이렇게 하면 제곱근 계산 비용을 줄일 수 있다.

## 볼록 껍질 문제(Convex-Hull Problem)

평면 또는 고차원 공간에서 주어진 점 집합의 **볼록 껍질(Convex Hull)**을 찾는 문제다.

### 문제 정의

**볼록 집합(Convex Set)**:

집합 내의 임의의 두 점 p, q를 연결하는 선분이 전부 집합에 속하는 경우

**볼록 껍질(Convex Hull)**:

점 집합 S를 포함하는 **가장 작은 볼록 집합**

### 시각적 이해

![Convex Hull 예시](/assets/img/posts/convex-hull.png)
> 
> - 8개의 점이 흩어져 있는 평면
> - 점들: p₁, p₂, p₃, p₄, p₅, p₆, p₇, p₈
> - 볼록 껍질: p₁, p₃, p₅, p₆, p₇을 연결한 다각형
> - 내부 점들(p₂, p₄, p₈)은 껍질에 포함되지 않음

- 삼각형의 볼록 껍질 → 삼각형 자체 (3개 꼭짓점)
- 원의 볼록 껍질 → 원 자체 (모든 둘레 점)

볼록 집합의 점 중에서 다른 두 점을 연결하는 선분의 중간 지점이 아닌 점(p₁, p₃, p₅, p₆, p₇)은 **극점(Extreme Point)**이다.

**예시**:
- 삼각형: 3개 꼭짓점이 극점
- 원: 둘레의 모든 점이 극점
- 다각형: 모든 꼭짓점이 극점


### 브루트 포스 알고리즘

**핵심 아이디어**:

두 점 $p_i$, $p_j$를 연결하는 선분이 볼록 껍질의 경계가 되려면:

→ **다른 모든 점들이 이 선분의 같은 쪽에 있어야 한다**

### 수학적 배경

**두 점을 지나는 직선의 방정식**:

점 $(x_1, y_1)$, $(x_2, y_2)$를 지나는 직선:

$$ax + by = c$$

여기서:
- $a = y_2 - y_1$
- $b = x_1 - x_2$  
- $c = x_1y_2 - y_1x_2$

**점이 직선의 어느 쪽에 있는지 판단**:

점 $(x, y)$에 대해 $ax + by - c$의 부호를 확인:
- $ax + by > c$ → 한쪽 반평면
- $ax + by < c$ → 반대쪽 반평면
- $ax + by = c$ → 직선 위

**알고리즘**:

1. 모든 점 쌍 $(p_i, p_j)$에 대해
2. 직선 방정식 계산
3. 다른 모든 점 $p_k$에 대해 $ax_k + by_k - c$ 계산
4. 모든 점이 같은 부호를 가지면 $(p_i, p_j)$는 경계 선분

### C++ 구현

```cpp
#include <vector>
using namespace std;

struct Point {
    double x, y;
};

// 세 점이 반시계 방향인지 확인 (외적 이용)
double crossProduct(const Point& O, const Point& A, const Point& B) {
    return (A.x - O.x) * (B.y - O.y) - (A.y - O.y) * (B.x - O.x);
}

vector<pair<int, int>> bruteForceConvexHull(const vector<Point>& points) {
    int n = points.size();
    vector<pair<int, int>> hull;
    
    // 모든 점 쌍 검사
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            bool isEdge = true;
            int sign = 0;  // 첫 번째 점의 부호 저장
            
            // 다른 모든 점이 같은 쪽에 있는지 확인
            for (int k = 0; k < n; k++) {
                if (k == i || k == j) continue;
                
                double cp = crossProduct(points[i], points[j], points[k]);
                
                if (cp != 0) {  // 직선 위에 있지 않은 경우
                    int currentSign = (cp > 0) ? 1 : -1;
                    
                    if (sign == 0) {
                        sign = currentSign;  // 첫 번째 부호 저장
                    } else if (sign != currentSign) {
                        isEdge = false;  // 다른 쪽에 점이 있음
                        break;
                    }
                }
            }
            
            if (isEdge) {
                hull.push_back({i, j});
            }
        }
    }
    
    return hull;
}
```

### 시간 복잡도 분석

**외부 루프**: $\frac{n(n-1)}{2}$개의 점 쌍

**내부 루프**: 각 쌍에 대해 $n-2$개의 점 검사

**총 연산 횟수**:

$$C(n) = \frac{n(n-1)}{2} \cdot (n-2) \approx \frac{n^3}{2}$$

**시간 복잡도**: **O(n³)**

### 더 효율적인 알고리즘

브루트 포스는 O(n³)이지만, 더 효율적인 알고리즘들이 존재한다:

- **Graham Scan**: O(n log n)
- **Jarvis March**: O(nh), h는 볼록 껍질의 꼭짓점 개수
- **Divide and Conquer**: O(n log n)

작은 점 집합에는 브루트 포스도 충분히 실용적이다.

## 브루트 포스의 한계와 개선

### 지금까지 다룬 알고리즘들의 복잡도

| 문제 | 브루트 포스 복잡도 | 개선 가능성 |
|-----|------------------|-----------|
| 선택 정렬 | Θ(n²) | 병합 정렬: O(n log n) |
| 버블 정렬 | Θ(n²) | 퀵 정렬: O(n log n) |
| 순차 탐색 | O(n) | 이진 탐색: O(log n) |
| 문자열 매칭 | O(nm) | KMP: O(n+m) |
| 최근접 쌍 | Θ(n²) | 분할 정복: O(n log n) |
| 볼록 껍질 | O(n³) | Graham Scan: O(n log n) |

### 브루트 포스를 사용해야 할 때

1. **문제 이해 단계**: 프로토타입으로 시작
2. **작은 입력**: n이 작으면 충분히 빠름
3. **일회성 작업**: 최적화 비용이 정당화되지 않음
4. **비교 기준**: 개선된 알고리즘의 성능 측정

### 다음 단계

브루트 포스의 한계를 극복하기 위한 알고리즘 설계 기법:
- **분할 정복(Divide and Conquer)**
- **동적 프로그래밍(Dynamic Programming)**
- **탐욕 알고리즘(Greedy Algorithm)**
- **백트래킹(Backtracking)**