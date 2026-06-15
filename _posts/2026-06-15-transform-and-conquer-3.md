---
title: "Transform-and-Conquer (3): 힙과 힙정렬, Horner의 법칙, 문제 축소"
date: 2026-06-15 18:00:00 +0900
categories: [Problem Solving Strategies, Transform and Conquer]
tags: [algorithm, transform-and-conquer, heap, heapsort, horner, binary-exponentiation, problem-reduction]
math: true
---


## 1. 우선순위 큐와 힙

우선순위 큐(priority queue)는 각 원소가 우선순위를 갖는 집합으로, 최고 우선순위 원소 찾기, 최고 우선순위 원소 삭제, 새 원소 추가라는 연산을 지원한다. 운영체제의 작업 스케줄링이나 통신망의 트래픽 관리 등에 쓰인다. 이 우선순위 큐를 효율적으로 구현하는 자료구조가 힙(heap)이다.

### 힙의 정의

힙은 노드마다 키를 하나씩 담은 이진 트리로, 다음 두 조건을 만족한다.

첫째는 모양 조건(shape property)이다. 트리가 본질적으로 완전(essentially complete)해야 한다. 즉 마지막 레벨을 제외한 모든 레벨이 꽉 차 있고, 마지막 레벨은 왼쪽부터 채워진다. 둘째는 부모 우위 조건(parental dominance, heap property)이다. 모든 노드의 키가 그 자식들의 키보다 크거나 같다(최대 힙 기준).

### 배열 표현: 표현 변경

힙의 강력함은 이 트리를 배열 하나로 표현할 수 있다는 데서 나온다. 트리 구조를 포인터 없이 배열 인덱스의 관계로 옮기는 것이므로, 전형적인 표현 변경이다. $1$부터 시작하는 인덱스를 쓰면 다음 관계가 성립한다.

$$\text{부모}(i) = \lfloor i/2 \rfloor, \qquad \text{왼쪽 자식}(i) = 2i, \qquad \text{오른쪽 자식}(i) = 2i+1$$

부모 우위 조건은 배열 언어로 다음과 같이 다시 쓸 수 있다.

$$H[i] \ge H[2i] \quad\text{그리고}\quad H[i] \ge H[2i+1] \qquad (1 \le i \le \lfloor n/2 \rfloor)$$

![힙의 트리 표현과 배열 표현](/assets/img/posts/heap-representation.png)
_같은 최대 힙을 트리(왼쪽)와 배열(오른쪽)로 나타낸 모습. 트리 구조를 포인터 없이 배열 인덱스 관계로 옮긴 것이 표현 변경이다. 인덱스 $i$의 자식은 $2i$와 $2i+1$에 있다_

---

## 2. 힙 구성 (Heap Construction)

주어진 리스트를 힙으로 만드는 방법은 두 가지다. 상향식(bottom-up)과 하향식(top-down)인데, 여기서는 더 효율적인 상향식을 본다.

### 상향식 힙 구성

배열을 주어진 순서 그대로 두고, 마지막 부모 노드($\lfloor n/2 \rfloor$ 위치)부터 루트까지 거꾸로 올라가며 각 노드에서 부모 우위 조건을 확인한다. 조건이 깨져 있으면 그 키를 더 큰 자식과 교환하고, 내려간 위치에서 다시 확인하기를 반복한다. 이 아래로 내려가는 보정을 sift-down(또는 reheap-down)이라 부른다.

```cpp
#include <vector>
#include <algorithm>
using namespace std;

// 1-indexed: H[1..n]
void siftDown(vector<int>& H, int i, int n) {
    while (true) {
        int largest = i, l = 2*i, r = 2*i + 1;
        if (l <= n && H[l] > H[largest]) largest = l;
        if (r <= n && H[r] > H[largest]) largest = r;
        if (largest == i) break;
        swap(H[i], H[largest]);
        i = largest;
    }
}

void buildHeap(vector<int>& H, int n) {
    for (int i = n/2; i >= 1; i--)   // 마지막 부모부터 루트까지
        siftDown(H, i, n);
}
```

### 복잡도

언뜻 각 노드에서 $O(\log n)$이 걸리니 $O(n \log n)$처럼 보이지만, 아래쪽 노드일수록 내려갈 수 있는 높이가 짧다는 점을 합산하면 더 낮은 경계가 나온다.

$$C(n) \le \sum_{i=1}^{n} \frac{n}{2^{i+1}} \cdot i \cdot 2 \in \Theta(n)$$

상향식 힙 구성은 선형 시간에 끝난다. 이 사실이 곧이어 볼 힙정렬의 효율을 뒷받침한다.

---

## 3. 힙에서 삭제와 힙정렬 (Heapsort)

### 최댓값 삭제

최대 힙에서 최댓값은 항상 루트($H[1]$)에 있다. 삭제는 루트와 마지막 원소를 교환하고 힙 크기를 하나 줄인 뒤, 새 루트에 sift-down을 한 번 적용해 부모 우위 조건을 복구한다. 한 번의 삭제는 $O(\log n)$이다.

### 힙정렬

B. R. Williams(1964)가 제안한 힙정렬은 두 단계로 진행된다. 먼저 배열을 힙으로 만들고($\Theta(n)$), 그다음 최댓값을 반복적으로 뽑아 배열 끝에서부터 채운다. 최댓값을 맨 뒤로 보내고 힙 크기를 줄이는 과정을 반복하면, 추가 메모리 없이 제자리(in-place)에서 오름차순 정렬이 완성된다.

```cpp
void heapSort(vector<int>& a) {       // a[1..n] 사용
    int n = a.size() - 1;
    buildHeap(a, n);                  // Θ(n)
    for (int last = n; last >= 2; last--) {
        swap(a[1], a[last]);          // 현재 최댓값을 끝으로
        siftDown(a, 1, last - 1);     // 남은 부분을 다시 힙으로  O(log n)
    }
}
```

### 복잡도

힙 구성이 $\Theta(n)$, 이후 $n-1$번의 삭제가 각각 $O(\log n)$이므로 전체는 다음과 같다.

$$T(n) = \underbrace{\Theta(n)}_{\text{구성}} + \underbrace{(n-1)\,O(\log n)}_{\text{삭제 반복}} = \Theta(n \log n)$$

| | 시간 (최악) | 추가 메모리 | 안정성 |
|---|:---:|:---:|:---:|
| 힙정렬 | $\Theta(n \log n)$ | $O(1)$ (제자리) | 불안정 |
| 병합 정렬 | $\Theta(n \log n)$ | $\Theta(n)$ | 안정 |
| 퀵 정렬 | $\Theta(n^2)$ (평균 $\Theta(n \log n)$) | $O(\log n)$ | 불안정 |

힙정렬은 최악의 경우에도 $\Theta(n \log n)$을 보장하면서 추가 메모리를 거의 쓰지 않는다는 점이 강점이다.

---

## 4. Horner의 법칙 (Horner's Rule)

### 다항식 평가 문제

차수 $n$의 다항식을 한 점 $x$에서 평가하는 문제다.

$$p(x) = a_n x^n + a_{n-1} x^{n-1} + \cdots + a_1 x + a_0$$

각 항을 따로 계산하면 거듭제곱에 많은 곱셈이 들어간다. 거듭제곱을 순진하게 계산할 경우 곱셈 횟수는 $\Theta(n^2)$에 이른다.

### 변환 아이디어: 중첩된 형태로

Horner의 법칙은 같은 다항식을 곱셈을 최소화하는 표현으로 다시 쓴다. $x$로 반복해서 묶어내는 것이다.

$$p(x) = (\cdots((a_n x + a_{n-1})x + a_{n-2})x + \cdots)x + a_0$$

이 중첩 형태를 안쪽에서 바깥쪽으로 평가하면, 각 단계마다 곱셈 한 번과 덧셈 한 번만 필요하다.

```cpp
#include <vector>
using namespace std;

// coef[0]=a_n, coef[1]=a_{n-1}, ..., coef[n]=a_0
double horner(const vector<double>& coef, double x) {
    double p = coef[0];
    for (size_t i = 1; i < coef.size(); i++)
        p = p * x + coef[i];
    return p;
}
```

곱셈과 덧셈이 각각 정확히 $n$번이므로 평가 비용은 $\Theta(n)$이다.

| 방법 | 곱셈 횟수 |
|---|:---:|
| 순진한 항별 계산 | $\Theta(n^2)$ |
| Horner의 법칙 | $n$ |

### 이진 거듭제곱

Horner의 사고방식은 거듭제곱 $a^k$를 빠르게 계산하는 데로 이어진다. 지수 $k$를 이진수로 보고, $a$를 제곱해 나가면서 해당 비트가 켜져 있을 때만 결과에 곱한다. 곱셈 횟수가 $\Theta(\log k)$로 줄어든다.

```cpp
long long fastPow(long long base, long long exp) {
    long long result = 1;
    while (exp > 0) {
        if (exp & 1) result *= base;   // 현재 비트가 1이면 곱한다
        base *= base;                  // base를 제곱
        exp >>= 1;                     // 다음 비트로
    }
    return result;
}
```

$a^k$를 단순히 $k-1$번 곱하면 $\Theta(k)$지만, 이진 거듭제곱은 $\Theta(\log k)$로 끝난다.

---

## 5. 문제 축소 (Problem Reduction)

변환 정복의 세 번째 변형은 문제를, 이미 알고리즘이 존재하는 다른 문제로 바꿔서 푸는 것이다. 실용적 가치를 가지려면 변환 비용과 다른 문제를 푸는 비용의 합이 원래 문제를 직접 푸는 것보다 효율적이어야 한다.

$$\text{문제 } P \;\xrightarrow{\;\text{변환}\;}\; \text{문제 } Q \;\xrightarrow{\;\text{기존 알고리즘}\;}\; Q\text{의 해} \;\Rightarrow\; P\text{의 해}$$

### 5.1 최소공배수

두 수의 최소공배수는 최대공약수 문제로 축소된다. 유클리드 호제법으로 최대공약수를 구하면 다음 관계로 최소공배수를 즉시 얻는다.

$$\text{lcm}(m, n) = \frac{m \cdot n}{\gcd(m, n)}$$

```cpp
long long gcd(long long a, long long b) { return b ? gcd(b, a % b) : a; }
long long lcm(long long a, long long b) { return a / gcd(a, b) * b; }
```

최소공배수를 소인수분해로 직접 구하는 것보다, 이미 빠른 알고리즘이 있는 최대공약수 문제로 옮기는 편이 훨씬 효율적이다.

### 5.2 그래프의 경로 수 세기

정점이 $n$개인 그래프에서 두 정점 사이의 길이 $k$인 보행(walk) 수를 세는 문제는 행렬 거듭제곱 문제로 축소된다. 인접 행렬을 $A$라 하면, $A^k$의 $(i, j)$ 원소가 곧 $i$에서 $j$로 가는 길이 $k$의 보행 수다.

$$(\text{$i$에서 $j$로 가는 길이 $k$ 보행 수}) = (A^k)_{ij}$$

예를 들어 삼각형 그래프(정점 $0, 1, 2$가 서로 연결)에서 $A^2$의 대각 원소는 $2$인데, 이는 각 정점에서 출발해 두 변을 거쳐 자기 자신으로 돌아오는 보행이 두 가지임을 뜻한다.

### 5.3 최적화 문제의 축소

최대화와 최소화는 부호만 뒤집으면 서로 변환된다. 따라서 최소화 알고리즘만 있으면 최대화 문제도 풀 수 있고, 그 반대도 성립한다.

$$\max_x f(x) = -\min_x \big(-f(x)\big)$$

### 5.4 그래프 문제로의 축소

여러 퍼즐과 게임은 그래프 문제로 축소해 풀 수 있다. 정점이 문제의 가능한 상태를 나타내고, 간선이 상태 사이의 허용된 전이를 나타내는 그래프를 상태 공간 그래프(state-space graph)라 한다. 초기 상태에서 목표 상태로 가는 경로를 찾는 문제가 되는 것이다.

농부가 늑대·염소·양배추를 배로 강 건너로 옮기는 고전 퍼즐이 대표적이다. 배에는 농부와 한 가지 물건만 탈 수 있고, 농부가 없으면 늑대가 염소를, 염소가 양배추를 먹는다. 각 상태(누가 어느 강둑에 있는지)를 정점으로, 허용된 한 번의 도강을 간선으로 놓으면, 퍼즐은 초기 상태에서 목표 상태까지의 경로 탐색으로 바뀐다. 이 퍼즐은 최소 일곱 번의 도강으로 푸는 두 가지 해를 가진다.



---
