---
title: "Transform-and-Conquer (1): 개념, Presorting, 가우스 소거법"
date: 2026-06-08 18:00:00 +0900
categories: [Problem Solving Strategies, Transform and Conquer]
tags: [algorithm, transform-and-conquer, presorting, gaussian-elimination, instance-simplification]
math: true
---



## 1. Transform-and-Conquer란?

Transform-and-Conquer(변환 정복)는 문제를 변환을 통해 해결하는 기법들의 집합이다. 두 단계로 진행된다. 먼저 변환 단계에서 문제의 인스턴스를 어떤 이유로든 풀기 더 쉬운 형태로 바꾸고, 이어지는 정복 단계에서 그 형태를 실제로 푼다.

무엇을 변환하느냐에 따라 세 가지 변형으로 나뉜다.

| 변형 | 변환 대상 | 설명 |
|---|---|---|
| 인스턴스 단순화 (instance simplification) | 같은 문제의 더 단순한 인스턴스 | 더 다루기 쉬운 형태로 바꿔 같은 문제를 푼다 |
| 표현 변경 (representation change) | 같은 인스턴스의 다른 표현 | 자료구조나 표현 방식을 바꾼다 |
| 문제 축소 (problem reduction) | 이미 알고리즘이 존재하는 다른 문제 | 다른 문제로 바꿔서 기존 해법을 재사용한다 |

이번 글에서는 인스턴스 단순화에 속하는 두 가지 대표적 기법인 Presorting과 가우스 소거법을 다룬다.

---

## 2. Presorting (사전 정렬)

### 핵심 아이디어

리스트에 관한 많은 문제들은 리스트가 정렬되어 있으면 훨씬 쉽게 풀린다. 그래서 "먼저 정렬한 뒤 문제를 푼다"는 전략 자체가 인스턴스 단순화의 전형이 된다. 탐색, 중앙값 계산(선택 문제), 원소 유일성 검사, 최빈값 계산 등이 정렬의 덕을 보는 대표적인 문제들이다.

이때 전체 알고리즘의 시간 효율은 사용하는 정렬 알고리즘의 효율에 좌우된다. 비교 기반 정렬의 하한이 $\Theta(n \log n)$이므로, presorting 기반 알고리즘은 보통 이 비용을 깔고 시작한다.

### 2.1 원소 유일성 검사

주어진 배열의 모든 원소가 서로 다른지 판별하는 문제다. 브루트 포스는 모든 쌍을 비교한다.

$$C_{worst}(n) = \sum_{i=0}^{n-2}\sum_{j=i+1}^{n-1} 1 = \frac{n(n-1)}{2} \in \Theta(n^2)$$

Presorting 기반 알고리즘은 배열을 먼저 정렬한 뒤, 인접한 원소만 한 번씩 비교해 같은 값이 붙어 있는지 확인한다.

```cpp
#include <vector>
#include <algorithm>
using namespace std;

bool uniqueBySorting(vector<int> a) {
    sort(a.begin(), a.end());                 // Θ(n log n)
    for (int i = 0; i + 1 < (int)a.size(); i++) // Θ(n)
        if (a[i] == a[i+1]) return false;
    return true;
}
```

정렬이 $\Theta(n \log n)$, 스캔이 $\Theta(n)$이므로 전체는 정렬에 지배된다.

$$T(n) = T_{sort}(n) + T_{scan}(n) = \Theta(n \log n) + \Theta(n) = \Theta(n \log n)$$

| 방법 | 시간 복잡도 |
|---|:---:|
| 브루트 포스 | $\Theta(n^2)$ |
| Presorting 기반 | $\Theta(n \log n)$ |

### 2.2 최빈값 계산

최빈값(mode)은 리스트에서 가장 자주 나타나는 값이다. 브루트 포스로 각 값의 빈도를 일일이 세면 $\Theta(n^2)$이 걸린다.

Presorting 기반 알고리즘의 아이디어는 단순하다. 정렬하면 같은 값들이 모두 인접하게 되므로, 한 번의 선형 스캔으로 가장 긴 동일 값 구간을 찾으면 된다.

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int modeBySorting(vector<int> a) {
    if (a.empty()) return -1;
    sort(a.begin(), a.end());                 // Θ(n log n)

    int modeVal = a[0], modeFreq = 1;
    int i = 0, n = a.size();
    while (i < n) {                            // Θ(n)
        int runVal = a[i], runLen = 1;
        while (i + 1 < n && a[i+1] == runVal) { i++; runLen++; }
        if (runLen > modeFreq) { modeFreq = runLen; modeVal = runVal; }
        i++;
    }
    return modeVal;
}
```

정렬 이후의 스캔은 선형 시간이므로, 전체 실행 시간은 정렬에 지배된다.

$$T(n) = \Theta(n \log n)$$

### 2.3 탐색 문제와 presorting의 가치

한 번만 탐색한다면 정렬할 필요 없이 순차 탐색($\Theta(n)$)이 더 낫다. 하지만 같은 리스트에 대해 반복적으로 탐색해야 한다면, 한 번 정렬($\Theta(n \log n)$)해두고 이후 각 질의를 이진 탐색($\Theta(\log n)$)으로 처리하는 편이 훨씬 효율적이다.

$$\underbrace{\Theta(n \log n)}_{\text{1회 정렬}} + \underbrace{q \cdot \Theta(\log n)}_{q\text{회 탐색}}$$

질의 수 $q$가 충분히 크면 presorting의 투자 비용이 회수된다.

### Presorting이 쓰이는 더 넓은 맥락

Presorting은 알고리즘 설계 전반에 깊이 스며들어 있다. 기하 알고리즘 다수가 사전 정렬에 의존하는데, 앞 챕터의 최근접 쌍 문제와 볼록 껍질 문제의 divide-and-conquer 알고리즘도 $x$좌표 기준 정렬을 전제로 했다. 방향성 비순환 그래프(DAG)에서는 위상 정렬을 먼저 하면 최장·최단 경로 같은 문제가 더 쉽게 풀린다. 또한 탐욕 기법(Chapter 9)에 기반한 대부분의 알고리즘은 입력의 사전 정렬을 본질적인 구성 요소로 요구한다.

---

## 3. 가우스 소거법 (Gaussian Elimination)

### 문제 정의

$n$개의 미지수에 대한 $n$개의 선형 방정식으로 이루어진 연립일차방정식을 푸는 문제다.

$$
\begin{cases}
a_{11}x_1 + a_{12}x_2 + \cdots + a_{1n}x_n = b_1 \\
a_{21}x_1 + a_{22}x_2 + \cdots + a_{2n}x_n = b_2 \\
\quad\quad\quad\quad\quad\vdots \\
a_{n1}x_1 + a_{n2}x_2 + \cdots + a_{nn}x_n = b_n
\end{cases}
\quad\Longleftrightarrow\quad A\mathbf{x} = \mathbf{b}
$$

### 변환의 아이디어: 상삼각 형태로

가우스 소거법은 이 시스템을 풀기 쉬운 동치 시스템으로 변환한다. 목표는 계수 행렬이 상삼각(upper-triangular) 형태인 $A'\mathbf{x} = \mathbf{b}'$다.

$$A\mathbf{x} = \mathbf{b} \quad\Longrightarrow\quad A'\mathbf{x} = \mathbf{b}'$$

상삼각 시스템은 맨 아래 방정식부터 위로 올라가며 후진 대입(back substitution)으로 즉시 풀 수 있다. 어려운 형태를 풀기 쉬운 형태로 바꾼다는 점에서 전형적인 인스턴스 단순화다.

### 기본 행 연산(elementary operations)

변환은 계수 행렬에 대한 다음 세 가지 기본 연산의 연쇄로 이루어지며, 이 연산들은 시스템의 해를 바꾸지 않는다. 두 방정식(행)을 서로 교환하거나, 한 방정식을 0이 아닌 상수배로 바꾸거나, 한 방정식을 그 방정식과 다른 방정식의 상수배의 합·차로 바꾸는 연산이다.

### 전진 소거(forward elimination)

$i$번째 단계에서, $i$행을 이용해 그 아래 모든 행의 $i$열 원소를 0으로 만든다. 각 행 $j > i$에 대해 다음을 수행한다.

$$\text{factor} = \frac{A[j][i]}{A[i][i]}, \qquad A[j] \leftarrow A[j] - \text{factor} \cdot A[i]$$

여기서 $A[i][i]$를 피벗(pivot)이라 부른다.

### 부분 피벗팅(partial pivoting)

피벗에는 두 가지 문제가 있다. $A[i][i] = 0$이면 0으로 나눌 수 없고, $A[i][i]$가 너무 작으면 반올림 오차에 의해 결과가 크게 왜곡될 수 있다.

이를 막기 위해 부분 피벗팅을 쓴다. $i$열에서 절댓값이 가장 큰 계수를 가진 행을 찾아 $i$행과 교환한 뒤, 그 새 원소를 피벗으로 사용한다. 행 교환은 기본 연산이므로 해를 바꾸지 않으면서 수치 안정성을 높인다.

### C++ 구현

```cpp
#include <iostream>
#include <vector>
#include <cmath>
using namespace std;

vector<double> gaussianElimination(vector<vector<double>> A, vector<double> b) {
    int n = A.size();
    for (int i = 0; i < n; i++) A[i].push_back(b[i]); // 증강 행렬 [A | b]

    // 전진 소거
    for (int i = 0; i < n; i++) {
        // 부분 피벗팅: i열에서 절댓값 최대인 행 선택
        int pivot = i;
        for (int j = i + 1; j < n; j++)
            if (fabs(A[j][i]) > fabs(A[pivot][i])) pivot = j;
        swap(A[i], A[pivot]);

        for (int j = i + 1; j < n; j++) {
            double factor = A[j][i] / A[i][i];
            for (int k = i; k <= n; k++)
                A[j][k] -= factor * A[i][k];
        }
    }

    // 후진 대입
    vector<double> x(n);
    for (int i = n - 1; i >= 0; i--) {
        double sum = A[i][n];
        for (int k = i + 1; k < n; k++) sum -= A[i][k] * x[k];
        x[i] = sum / A[i][i];
    }
    return x;
}

int main() {
    // 2x +  y -  z =  8
    // -3x - y + 2z = -11
    // -2x + y + 2z = -3   => x=2, y=3, z=-1
    vector<vector<double>> A = { {2,1,-1}, {-3,-1,2}, {-2,1,2} };
    vector<double> b = { 8, -11, -3 };
    vector<double> x = gaussianElimination(A, b);
    for (double v : x) cout << v << " ";  // 2 3 -1
    cout << "\n";
    return 0;
}
```

### 복잡도 분석

기본 연산을 곱셈으로 잡는다. 전진 소거에서 $i$번째 단계는 약 $(n-i)(n-i+1)$번의 곱셈을 수행하며, 전체를 합산하면 다음과 같다.

$$M(n) = \sum_{i=1}^{n-1} (n-i)(n-i+1) \approx \frac{n^3}{3} \in \Theta(n^3)$$

후진 대입은 $\Theta(n^2)$이므로 전체는 전진 소거에 지배된다.

$$T(n) \in \Theta(n^3)$$

### 더 넓은 활용

가우스 소거법으로 만든 상삼각 구조는 다양한 선형대수 연산의 기반이 된다. 행렬을 하삼각 $L$과 상삼각 $U$의 곱으로 분해하는 LU 분해는 같은 $A$에 대해 여러 $\mathbf{b}$를 풀 때 효율적이다. $A\mathbf{x} = \mathbf{e}_i$를 각 단위 벡터에 대해 풀면 역행렬의 열을 얻을 수 있고, 상삼각 행렬의 행렬식은 대각 원소들의 곱으로 간단히 구해진다.

---


