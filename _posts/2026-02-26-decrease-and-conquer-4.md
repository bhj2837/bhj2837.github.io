---
title: "Decrease-and-Conquer (4): 조합적 객체 생성 알고리즘"
date: 2026-02-26 18:00:00 +0900
categories: [Problem Solving Strategies, Decrease and Conquer]
tags: [algorithm, decrease-and-conquer, permutation, subset, johnson-trotter, gray-code]
math: true
---

지금까지 Decrease-and-Conquer의 세 가지 변형(Decrease-by-Constant, Decrease-by-Constant-Factor, Variable-Size-Decrease)을 각각 살펴봤다. 이번 포스트는 Decrease-and-Conquer 시리즈의 마지막으로, **조합적 객체(Combinatorial Objects)를 생성하는 알고리즘**을 다룬다.

---

## 조합적 객체란?

알고리즘에서 자주 등장하는 조합적 객체의 대표적인 세 가지 유형은 다음과 같다.

| 종류 | 설명 | 개수 |
|---|---|---|
| 순열 (Permutation) | $n$개 원소를 나열하는 모든 방법 | $n!$ |
| 조합 (Combination) | $n$개 중 $k$개를 고르는 방법 | $\binom{n}{k}$ |
| 부분집합 (Subset) | $n$개 원소 집합의 모든 부분집합 | $2^n$ |

TSP나 배낭 문제처럼 여러 선택지를 탐색해야 하는 문제에서는 이 조합적 객체들을 직접 열거해야 한다. 이 포스트에서는 순열과 부분집합을 생성하는 알고리즘에 집중한다.

---

## 1. 순열 생성 (Generating Permutations)

### 기본 아이디어: Decrease-by-One

$\{1, 2, \ldots, n\}$의 모든 순열을 생성하는 문제에 Decrease-by-One 전략을 적용하면 다음과 같이 생각할 수 있다.

> $\{1, \ldots, n-1\}$의 모든 $(n-1)!$개 순열을 이미 알고 있다고 가정하자.  
> 그러면 각 순열의 $n$개 위치에 $n$을 삽입하여 크기 $n$의 순열을 만들 수 있다.

각 $(n-1)!$개의 순열에 $n$개의 위치가 있으므로, 전체 순열 수는 $n \cdot (n-1)! = n!$개가 된다.

### Bottom-up 생성 과정

이 아이디어를 Bottom-up 방식으로 구체화하면, 작은 순열부터 쌓아 올리는 방식이 된다. $n = 3$인 경우를 예로 들면:

| 단계 | 동작 | 생성된 순열 |
|---|---|---|
| 시작 | $\{1\}$ | `1` |
| $n=2$ 삽입 | `1`에 2를 오른쪽→왼쪽으로 삽입 | `12`, `21` |
| $n=3$ 삽입 (12에) | `12`에 3을 오른쪽→왼쪽으로 삽입 | `123`, `132`, `312` |
| $n=3$ 삽입 (21에) | `21`에 3을 왼쪽→오른쪽으로 삽입 | `321`, `231`, `213` |

삽입 방향을 매 순열마다 교대(오른쪽→왼쪽, 왼쪽→오른쪽)로 바꾸면, 연속된 두 순열이 인접한 두 원소의 교환만으로 달라지는 성질이 생긴다. 이를 **최소 변화 요건(minimal-change requirement)**이라 부른다.

### 최소 변화 요건 (Minimal-Change Requirement)

최소 변화 요건이란 연속된 두 순열이 **인접한 두 원소의 교환 딱 한 번**으로 달라지는 것이다.

이 성질이 중요한 이유는 **실용적인 효율성** 때문이다. TSP를 예로 들면, 순열이 하나 바뀔 때 투어 경로에서 달라지는 간선이 단 두 개이므로 새 투어의 비용을 이전 비용에서 **상수 시간** $O(1)$만에 갱신할 수 있다.

| 투어 | 비용 계산 |
|---|---|
| a-b-c-d-e-f-**g-h**-a | $2+4+3+5+2+1+4+1 = 22$ |
| a-b-c-d-e-f-**h-g**-a | $2+4+3+5+2+3+4+2 = 25$ |

두 투어에서 차이가 나는 간선은 f-g, g-h, h-a 부분뿐이므로, 전체를 다시 계산하지 않아도 된다.

### Johnson-Trotter 알고리즘

최소 변화 요건을 만족하면서 모든 $n!$개 순열을 생성하는 알고리즘이 **Johnson-Trotter 알고리즘**이다. Bottom-up처럼 작은 $n$부터 반복적으로 생성하지 않고, **방향(direction)**이라는 개념을 도입해 단일 루프로 처리한다.

#### 핵심 개념: 이동 가능한 원소 (Mobile Element)

각 원소 $k$에는 화살표 방향이 부여된다(처음에는 모두 왼쪽). 원소 $k$가 **이동 가능(mobile)**하려면, 화살표가 가리키는 방향의 **이웃 원소가 $k$보다 작아야** 한다.

예를 들어 $\overleftarrow{3}\ \overleftarrow{2}\ \overrightarrow{4}\ \overleftarrow{1}$ 배열에서:

| 원소 | 화살표 방향 | 가리키는 이웃 | 이동 가능 여부 |
|---|---|---|---|
| 3 | ← (왼쪽) | 없음 (가장 왼쪽) | ✗ |
| 2 | ← (왼쪽) | 3 | ✗ (2 < 3이 아님) |
| 4 | → (오른쪽) | 1 | ✓ (1 < 4) |
| 1 | ← (왼쪽) | 없음 | ✗ |

따라서 이 배열에서 이동 가능한 원소는 3과 4이다 (3도 오른쪽 이웃 2가 3보다 작으므로 이동 가능).

#### 알고리즘 동작

1. 초기 순열을 $\overleftarrow{1}\ \overleftarrow{2}\ \cdots\ \overleftarrow{n}$으로 설정한다.
2. 이동 가능한 원소 중 **가장 큰 원소 $k$**를 찾는다.
3. $k$를 화살표가 가리키는 인접 원소와 교환한다.
4. $k$보다 큰 모든 원소의 화살표 방향을 반전시킨다.
5. 새 순열을 목록에 추가하고, 이동 가능한 원소가 없을 때까지 반복한다.

#### $n = 3$ 전체 과정

| 순열 | 가장 큰 mobile | 동작 |
|---|---|---|
| $\overleftarrow{1}\ \overleftarrow{2}\ \overleftarrow{3}$ | 3 (←) | 3을 왼쪽으로 이동 |
| $\overleftarrow{1}\ \overleftarrow{3}\ \overleftarrow{2}$ | 3 (←) | 3을 왼쪽으로 이동 |
| $\overleftarrow{3}\ \overleftarrow{1}\ \overleftarrow{2}$ | 3 (←) | 3이 왼쪽 끝 → 이동 불가, 2가 mobile → 2 이동, 3의 방향 반전(→) |
| $\overleftarrow{3}\ \overleftarrow{2}\ \overleftarrow{1}$ | → 3 (→) | 3을 오른쪽으로 이동 |
| $\overleftarrow{2}\ \overleftarrow{3}\ \overleftarrow{1}$ | → 3 (→) | 3을 오른쪽으로 이동 |
| $\overleftarrow{2}\ \overleftarrow{1}\ \overleftarrow{3}$ | 이동 가능한 원소 없음 | **종료** |

생성 순서: `123` → `132` → `312` → `321` → `231` → `213`  
연속된 두 순열은 항상 인접 원소 하나의 교환으로만 다르다.

#### C++ 구현

```cpp
#include <iostream>
#include <vector>
using namespace std;

void johnsonTrotter(int n) {
    vector<int> perm(n);
    vector<int> dir(n, -1); // -1: 왼쪽, +1: 오른쪽
    for (int i = 0; i < n; i++) perm[i] = i + 1;

    // 초기 순열 출력
    for (int x : perm) cout << x << " ";
    cout << "\n";

    while (true) {
        // 이동 가능한 가장 큰 원소 찾기
        int mobileIdx = -1;
        for (int i = 0; i < n; i++) {
            int j = i + dir[i]; // 화살표가 가리키는 위치
            if (j >= 0 && j < n && perm[j] < perm[i]) {
                if (mobileIdx == -1 || perm[i] > perm[mobileIdx])
                    mobileIdx = i;
            }
        }
        if (mobileIdx == -1) break; // 이동 가능한 원소 없음 → 종료

        int mobileVal = perm[mobileIdx];
        int swapIdx = mobileIdx + dir[mobileIdx];

        // 교환
        swap(perm[mobileIdx], perm[swapIdx]);
        swap(dir[mobileIdx], dir[swapIdx]);

        // mobileVal보다 큰 원소의 방향 반전
        for (int i = 0; i < n; i++)
            if (perm[i] > mobileVal)
                dir[i] = -dir[i];

        for (int x : perm) cout << x << " ";
        cout << "\n";
    }
}
```

### 복잡도 분석

Johnson-Trotter 알고리즘은 $n!$개의 순열을 생성하며, 각 순열을 만드는 데 $O(n)$의 작업이 필요하다. 따라서 전체 복잡도는 $\Theta(n \cdot n!)$이다. 하지만 이 복잡도는 알고리즘 자체의 비효율이 아니라, **생성해야 할 순열의 수 자체가 $n!$개**이기 때문이다. 출력만 해도 $\Theta(n \cdot n!)$이 필요하므로, 알고리즘 자체는 최적에 가깝다.

| n | n! | 시간 |
|---|---|---|
| 5 | 120 | 즉시 |
| 10 | 3,628,800 | 약 1초 |
| 12 | 479,001,600 | 수 분 |
| 15 | 1조 이상 | 현실적으로 불가 |

$n$이 조금만 커져도 처리가 불가능하다는 점에서, 순열 전체를 열거하는 방식은 작은 $n$에서만 실용적이다.

---

## 2. 부분집합 생성 (Generating Subsets)

### 기본 아이디어: Decrease-by-One

집합 $A = \{a_1, \ldots, a_n\}$의 모든 $2^n$개 부분집합(멱집합, power set)을 생성하는 문제에도 Decrease-by-One을 적용할 수 있다.

$A$의 모든 부분집합을 두 그룹으로 나눈다.

- $a_n$을 **포함하지 않는** 부분집합 → $\{a_1, \ldots, a_{n-1}\}$의 부분집합과 동일
- $a_n$을 **포함하는** 부분집합 → $\{a_1, \ldots, a_{n-1}\}$의 각 부분집합에 $a_n$을 추가한 것

따라서 $\{a_1, \ldots, a_{n-1}\}$의 부분집합 목록을 알고 있다면, 각 원소에 $a_n$을 추가하는 것만으로 크기 $n$ 집합의 부분집합 전체를 얻을 수 있다.

### Bottom-up 생성 과정

| $n$ | 생성되는 부분집합 |
|---|---|
| 0 | $\emptyset$ |
| 1 | $\emptyset,\ \{a_1\}$ |
| 2 | $\emptyset,\ \{a_1\},\ \{a_2\},\ \{a_1, a_2\}$ |
| 3 | $\emptyset,\ \{a_1\},\ \{a_2\},\ \{a_1,a_2\},\ \{a_3\},\ \{a_1,a_3\},\ \{a_2,a_3\},\ \{a_1,a_2,a_3\}$ |

$n=2$ 목록 뒤에 각 원소에 $a_3$을 추가한 목록을 이어 붙이면 $n=3$ 목록이 된다. 매 단계마다 이전 목록의 크기가 두 배가 되므로, 전체 생성 비용은 $\Theta(2^n)$이다.

### 비트 문자열을 이용한 부분집합 표현

부분집합을 표현하는 더 편리한 방법은 **비트 문자열(bit string)**을 이용하는 것이다. $n$개 원소 집합의 각 부분집합은 길이 $n$의 비트 문자열과 일대일로 대응된다.

> $b_i = 1$이면 $a_i$가 부분집합에 포함, $b_i = 0$이면 포함되지 않음

$n = 3$인 경우:

| 비트 문자열 | 대응 부분집합 |
|---|---|
| `000` | $\emptyset$ |
| `001` | $\{a_3\}$ |
| `010` | $\{a_2\}$ |
| `011` | $\{a_2, a_3\}$ |
| `100` | $\{a_1\}$ |
| `101` | $\{a_1, a_3\}$ |
| `110` | $\{a_1, a_2\}$ |
| `111` | $\{a_1, a_2, a_3\}$ |

이 방법은 $0$부터 $2^n - 1$까지의 정수를 이진수로 나열하는 것과 같다. 구현이 매우 간단하고, 비트 연산으로 포함 여부를 $O(1)$에 확인할 수 있어 실무에서 가장 많이 쓰인다.

#### C++ 구현 (비트마스크)

```cpp
#include <iostream>
#include <vector>
using namespace std;

// 집합 {a[0], ..., a[n-1]}의 모든 부분집합 출력
void generateSubsets(vector<int>& a) {
    int n = a.size();
    for (int mask = 0; mask < (1 << n); mask++) {
        cout << "{ ";
        for (int i = 0; i < n; i++) {
            if (mask & (1 << i))   // i번째 비트가 1이면 포함
                cout << a[i] << " ";
        }
        cout << "}\n";
    }
}
```

### 최소 변화 부분집합: 이진 반사 그레이 코드 (Binary Reflected Gray Code)

비트 문자열을 순서대로 나열할 때 한 가지 문제가 있다. `011`에서 `100`으로 넘어갈 때처럼 여러 비트가 동시에 바뀌는 경우가 생긴다. 순열의 최소 변화 요건처럼, 부분집합에서도 **연속된 두 비트 문자열이 단 하나의 비트만 다른** 순서를 만들 수 있을까?

이 질문의 답이 바로 **이진 반사 그레이 코드(Binary Reflected Gray Code, BRGC)**다.

$n = 3$의 BRGC:

```
000 → 001 → 011 → 010 → 110 → 111 → 101 → 100
```

연속된 두 코드는 정확히 1비트만 다르다. 또한 마지막 코드(`100`)와 첫 번째 코드(`000`)도 1비트만 다르므로, BRGC는 **순환적(cyclic)**이다.

### BRGC 생성 알고리즘 (Decrease-by-One)

BRGC는 재귀적으로 구성할 수 있다.

1. $n = 1$이면 $[0, 1]$을 반환한다.
2. $n > 1$이면 $BRGC(n-1)$로 목록 $L_1$을 생성한다.
3. $L_1$을 역순으로 복사하여 $L_2$를 만든다.
4. $L_1$의 각 코드 앞에 `0`을 붙인다.
5. $L_2$의 각 코드 앞에 `1`을 붙인다.
6. $L_1$과 $L_2$를 이어 붙인 것이 $BRGC(n)$이다.

$n = 2$ 생성 과정:

| 단계 | 내용 |
|---|---|
| $BRGC(1)$ | $[0,\ 1]$ |
| $L_1$ | $[0,\ 1]$, 앞에 0 붙임 → $[00,\ 01]$ |
| $L_2$ | $[1,\ 0]$ (역순), 앞에 1 붙임 → $[11,\ 10]$ |
| $BRGC(2)$ | $[00,\ 01,\ 11,\ 10]$ |

$n = 3$ 생성 과정:

| 단계 | 내용 |
|---|---|
| $BRGC(2)$ | $[00,\ 01,\ 11,\ 10]$ |
| $L_1$ | 앞에 0 붙임 → $[000,\ 001,\ 011,\ 010]$ |
| $L_2$ | $BRGC(2)$ 역순 + 앞에 1 붙임 → $[110,\ 111,\ 101,\ 100]$ |
| $BRGC(3)$ | $[000,\ 001,\ 011,\ 010,\ 110,\ 111,\ 101,\ 100]$ |

$L_1$의 마지막 코드(`010`)와 $L_2$의 첫 번째 코드(`110`)는 맨 앞 비트만 다르다. 이 덕분에 이어 붙여도 최소 변화 성질이 유지된다.

#### C++ 구현

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

vector<string> brgc(int n) {
    if (n == 1) return {"0", "1"};

    vector<string> L1 = brgc(n - 1);
    vector<string> L2(L1.rbegin(), L1.rend()); // L1의 역순

    for (auto& s : L1) s = "0" + s; // L1 앞에 0 붙임
    for (auto& s : L2) s = "1" + s; // L2 앞에 1 붙임

    L1.insert(L1.end(), L2.begin(), L2.end());
    return L1;
}

int main() {
    int n = 3;
    vector<string> gray = brgc(n);
    for (const string& code : gray)
        cout << code << "\n";
    return 0;
}
```

실행 결과 (`n = 3`):

```
000
001
011
010
110
111
101
100
```

---

## 두 알고리즘 비교 요약

| 항목 | 순열 생성 (Johnson-Trotter) | 부분집합 생성 (BRGC) |
|---|---|---|
| 대상 | $\{1, \ldots, n\}$의 모든 순열 | $A$의 모든 부분집합 |
| 생성 개수 | $n!$ | $2^n$ |
| Decrease-and-Conquer 유형 | Decrease-by-One | Decrease-by-One |
| 최소 변화 성질 | 인접한 두 원소 교환 | 비트 1개 변경 |
| 시간 복잡도 | $\Theta(n \cdot n!)$ | $\Theta(n \cdot 2^n)$ |
| 핵심 아이디어 | mobile element + 방향 | 반사(reflection) + 0/1 접두사 |

두 알고리즘 모두 Decrease-by-One 아이디어에서 출발하여, 최소 변화 요건을 만족하는 효율적인 열거 순서를 만들어낸다는 공통점이 있다.

---

## Decrease-and-Conquer 시리즈 전체 요약

4편에 걸쳐 Decrease-and-Conquer의 모든 내용을 다뤘다. 전체를 한눈에 정리하면 다음과 같다.

| 포스트 | 변형 | 다룬 알고리즘 |
|---|---|---|
| 1편 | Decrease-by-One | 삽입 정렬, 위상 정렬, GCD 맛보기 |
| 2편 | Decrease-by-Constant-Factor | 이진 탐색, 가짜 동전, 러시아 농부 곱셈, 요세푸스 |
| 3편 | Variable-Size-Decrease | Quickselect, 보간 탐색, BST |
| 4편 | Decrease-by-One (조합적 객체) | Johnson-Trotter (순열), BRGC (부분집합) |

다음 포스트에서는 **Divide-and-Conquer**를 다룬다. Decrease-and-Conquer가 문제를 부분 문제 **하나**로 줄인다면, Divide-and-Conquer는 문제를 **여러 개**의 부분 문제로 나눠 각각 해결한 뒤 결합하는 전략이다. 대표적인 알고리즘으로 병합 정렬, 퀵 정렬, 이진 탐색 트리 구성 등이 있다.
