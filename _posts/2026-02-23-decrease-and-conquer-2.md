---
title: "Decrease-and-Conquer (2): Decrease-by-Constant-Factor"
date: 2026-02-23 18:00:00 +0900
categories: [Problem Solving Strategies, Decrease and Conquer]
tags: [algorithm, decrease-and-conquer, binary-search, fake-coin, russian-peasant, josephus]
math: true
---


## Decrease-by-Constant-Factor란?

Decrease-by-Constant-Factor는 각 재귀 단계에서 문제의 크기를 **상수 배(constant factor)**만큼 줄이는 전략이다. 가장 흔한 경우는 크기를 절반으로 줄이는 **Decrease-by-Half**이며, 이 경우 점화식의 형태는 다음과 같다.

$$T(n) = T\!\left(\left\lfloor n/2 \right\rfloor\right) + f(n)$$

$f(n)$이 $O(1)$ 혹은 $O(n)$ 수준이라면 전체 복잡도는 각각 $\Theta(\log n)$, $\Theta(n)$이 된다. 이 포스트에서는 다음 네 가지 알고리즘을 순서대로 살펴본다.

| 알고리즘 | 핵심 아이디어 | 복잡도 |
|---|---|---|
| Binary Search | 탐색 범위를 절반으로 줄임 | $\Theta(\log n)$ |
| Fake-Coin Problem | 동전 무더기를 절반으로 나눠 저울질 | $\Theta(\log n)$ |
| Russian Peasant Multiplication | $n$을 절반씩 줄이며 $m$을 두 배씩 늘림 | $\Theta(\log n)$ |
| Josephus Problem | 원형 구조를 절반으로 줄여 생존자 위치 계산 | $\Theta(\log n)$ |

---

## 1. 이진 탐색 (Binary Search)

### 개념

이진 탐색은 정렬된 배열에서 탐색 키 $K$를 찾는 알고리즘이다. 매 단계마다 배열의 중앙 원소 $A[m]$과 $K$를 비교하여 탐색 범위를 절반으로 좁혀나간다.

- $K = A[m]$이면 탐색 성공
- $K < A[m]$이면 왼쪽 절반을 탐색
- $K > A[m]$이면 오른쪽 절반을 탐색

```
A[0] ... A[m-1] | A[m] | A[m+1] ... A[n-1]
  K < A[m]이면 ←         K > A[m]이면 →
```

이는 전형적인 Decrease-by-Half 구조다. 크기 $n$의 문제를 크기 $\lfloor n/2 \rfloor$의 부분 문제 하나로 줄인다는 점에서 Decrease-and-Conquer임을 확인할 수 있다.

### 동작 예시: K = 70

배열: `[3, 14, 27, 31, 39, 42, 55, 70, 74, 81, 85, 93, 98]` (인덱스 0~12)

| 반복 | l | m | r | A[m] | 비교 결과 |
|---|---|---|---|---|---|
| 1 | 0 | 6 | 12 | 55 | 70 > 55 → 오른쪽 탐색 |
| 2 | 7 | 9 | 12 | 81 | 70 < 81 → 왼쪽 탐색 |
| 3 | 7 | 7 | 8 | 70 | 70 = 70 → **탐색 성공** |

3번의 비교만으로 인덱스 7의 원소를 찾았다.

### 복잡도 분석

기본 연산은 키 비교(key comparison)다.

$$C(n) = C\!\left(\left\lfloor n/2 \right\rfloor\right) + 1, \quad C(1) = 1$$

이를 풀면 $C(n) = \lfloor \log_2 n \rfloor + 1 \in \Theta(\log n)$이다.

| 케이스 | 복잡도 | 설명 |
|---|---|---|
| 최선 | $\Theta(1)$ | 첫 번째 비교에서 $K = A[m]$ |
| 평균 | $\Theta(\log n)$ | 탐색 성공/실패 평균 |
| 최악 | $\Theta(\log n)$ | 탐색 실패 또는 끝까지 탐색 |

순차 탐색의 $\Theta(n)$과 비교했을 때, $n = 10^6$이라면 최대 20번의 비교로 탐색을 끝낼 수 있다는 의미다.

### C++ 구현 (비재귀)

```cpp
// 비재귀(Bottom-up) 이진 탐색
// 입력: 오름차순 정렬된 배열 A, 크기 n, 탐색 키 K
// 출력: K의 인덱스, 없으면 -1
int binarySearch(int A[], int n, int K) {
    int l = 0, r = n - 1;
    while (l <= r) {
        int m = (l + r) / 2;
        if (K == A[m])
            return m;
        else if (K < A[m])
            r = m - 1;
        else
            l = m + 1;
    }
    return -1;
}
```

재귀 버전도 작성할 수 있다.

```cpp
// 재귀(Top-down) 이진 탐색
int binarySearchRecursive(int A[], int l, int r, int K) {
    if (l > r) return -1;
    int m = (l + r) / 2;
    if (K == A[m])      return m;
    else if (K < A[m])  return binarySearchRecursive(A, l, m - 1, K);
    else                return binarySearchRecursive(A, m + 1, r, K);
}
```

비재귀 버전이 함수 호출 오버헤드가 없으므로 실무에서는 더 선호된다.

---

## 2. 가짜 동전 문제 (Fake-Coin Problem)

### 문제 정의

$n$개의 동전 중 하나가 진짜보다 가벼운 가짜 동전이다. 양팔 저울(balance scale)을 이용해 두 그룹을 동시에 비교할 수 있을 때, 가짜 동전을 찾는 최소 횟수를 구하라.

### 알고리즘 아이디어

$n$개의 동전을 두 그룹 $\lfloor n/2 \rfloor$개씩으로 나누고 나머지 1개(n이 홀수일 때)를 별도로 둔다.

| 저울 결과 | 다음 행동 |
|---|---|
| 왼쪽이 가벼움 | 왼쪽 그룹 안에 가짜 동전 → 왼쪽 그룹에 동일한 방법 적용 |
| 오른쪽이 가벼움 | 오른쪽 그룹 안에 가짜 동전 → 오른쪽 그룹에 동일한 방법 적용 |
| 양쪽이 같음 | 별도로 뺀 1개가 가짜 동전 → 즉시 확인 |

### 점화식과 복잡도

최악의 경우(매번 두 그룹의 무게가 다른 경우) 저울질 횟수 $W(n)$에 대한 점화식은 다음과 같다.

$$W(n) = W\!\left(\left\lfloor n/2 \right\rfloor\right) + 1 \quad (n > 1), \quad W(1) = 0$$

이를 풀면 $W(n) = \lfloor \log_2 n \rfloor \in \Theta(\log n)$이다.

예를 들어 동전이 1000개라면, 저울질을 최대 9번만 하면 가짜 동전을 찾아낼 수 있다.

| n (동전 수) | 최악 저울질 횟수 |
|---|---|
| 2 | 1 |
| 4 | 2 |
| 8 | 3 |
| 16 | 4 |
| 1024 | 10 |

### Decrease-and-Conquer 관점

매 단계마다 고려 대상 동전의 수가 절반이 된다는 점에서 전형적인 Decrease-by-Half 구조다. 이진 탐색과 구조적으로 매우 유사하며, 차이는 탐색 키 대신 저울의 무게 비교를 사용한다는 점이다.

### C++ 구현

```cpp
// fake = 가짜 동전의 인덱스 (0-indexed, 실제로는 미지수)
// 여기서는 시뮬레이션: coins[i] = 1(진짜), 0(가짜)
// 가짜 동전의 인덱스를 반환
int findFakeCoin(std::vector<int>& coins, int l, int r) {
    if (l == r) return l; // 동전이 1개면 그게 가짜
    int mid = (l + r) / 2;
    int leftSize  = mid - l + 1;
    int rightSize = r - mid;

    // 왼쪽/오른쪽 그룹 무게 합산 (저울 시뮬레이션)
    int leftSum = 0, rightSum = 0;
    for (int i = l; i <= mid; i++)      leftSum  += coins[i];
    for (int i = mid + 1; i <= r; i++)  rightSum += coins[i];

    if (leftSize == rightSize && leftSum == rightSum) {
        // 두 그룹 크기가 같은데 무게도 같다면: n이 홀수일 때 별도 동전이 가짜
        // (이 예제에서는 별도 동전 처리를 생략하고 재귀 구조만 보임)
        return -1;
    }
    // 더 가벼운(합이 작은) 쪽에 가짜 동전 존재
    if (leftSum < rightSum)
        return findFakeCoin(coins, l, mid);
    else
        return findFakeCoin(coins, mid + 1, r);
}
```

---

## 3. 러시아 농부 곱셈법 (Russian Peasant Multiplication)

### 문제 정의

두 양의 정수 $n$과 $m$의 곱 $n \cdot m$을 곱셈 연산 없이 절반/두배/덧셈만으로 계산하라.

### 핵심 공식

$$n \cdot m = \begin{cases} \dfrac{n}{2} \cdot 2m & (n \text{이 짝수}) \\[6pt] \dfrac{n-1}{2} \cdot 2m + m & (n \text{이 홀수}) \end{cases}$$

$n = 1$이 되면 $1 \cdot m = m$으로 종료된다. 즉, 세 가지 연산만 사용한다.

| 연산 | 역할 |
|---|---|
| $n \leftarrow \lfloor n/2 \rfloor$ | 문제 크기를 절반으로 줄임 (right shift) |
| $m \leftarrow 2m$ | $m$을 두 배로 늘림 (left shift) |
| 누적 덧셈 | $n$이 홀수일 때 현재 $m$을 결과에 더함 |

### 동작 예시: 50 × 65

| $n$ | $m$ | n 홀수 여부 | 누적 합 |
|---|---|---|---|
| 50 | 65 | 짝수 | 0 |
| 25 | 130 | **홀수** | 130 |
| 12 | 260 | 짝수 | 130 |
| 6 | 520 | 짝수 | 130 |
| 3 | 1040 | **홀수** | 1170 |
| 1 | 2080 | **홀수** | 3250 |

$n = 1$에서 종료. $50 \times 65 = 3250$ ✓

$n$이 홀수인 행의 $m$ 값(130, 1040, 2080)을 모두 더하면 $130 + 1040 + 2080 = 3250$이 된다.

이 방법은 이진법(binary representation)과 밀접한 연관이 있다. $n = 50 = (110010)_2$이므로, 1인 비트 위치에 해당하는 $m$ 값들을 더하는 것과 동일하다.

$$50 = 32 + 16 + 2, \quad 50 \times 65 = 32 \times 65 + 16 \times 65 + 2 \times 65$$

### 복잡도 분석

$n$이 절반씩 줄어드므로 반복 횟수는 $\lfloor \log_2 n \rfloor + 1$번이다. 각 단계에서 연산이 $O(1)$이므로 전체 복잡도는 $\Theta(\log n)$이다.

### C++ 구현

```cpp
// Russian Peasant Multiplication: n * m 계산
// 곱셈 연산 없이 오직 shift와 덧셈만 사용
long long russianPeasant(long long n, long long m) {
    long long result = 0;
    while (n >= 1) {
        if (n % 2 == 1)   // n이 홀수이면 m을 결과에 누적
            result += m;
        n >>= 1;          // n을 절반으로 줄임 (right shift)
        m <<= 1;          // m을 두 배로 늘림 (left shift)
    }
    return result;
}
```

---

## 4. 요세푸스 문제 (Josephus Problem)

### 문제 정의

$n$명이 원형으로 서서 1번부터 번호를 매긴다. 1번부터 시작하여 **한 명씩 건너뛰며(두 번째마다)** 제거할 때, 마지막으로 살아남는 사람의 번호 $J(n)$을 구하라.

예를 들어 $n = 6$이면 제거 순서는 2, 4, 6, 3, 1이고 생존자는 5번 → $J(6) = 5$이다.  
$n = 7$이면 제거 순서는 2, 4, 6, 1, 5, 3이고 생존자는 7번 → $J(7) = 7$이다.

### 점화식

$n = 2^m + L$ ($0 \le L < 2^m$)으로 표현할 때:

$$J(2^m + L) = 2L + 1$$

또는 재귀적 점화식으로 표현하면:

$$J(1) = 1$$
$$J(2n) = 2J(n) - 1$$
$$J(2n+1) = 2J(n) + 1$$

짝수 $n$의 경우: 첫 번째 라운드에서 짝수 번호가 모두 제거되고 홀수 번호만 남는다. 이 상태는 $n/2$명의 새로운 요세푸스 문제가 된다.

| n | J(n) | n의 이진 표현 | 계산 |
|---|---|---|---|
| 1 | 1 | $1$ | $2 \cdot 0 + 1 = 1$ |
| 2 | 1 | $10$ | $2 \cdot 1 - 1 = 1$ |
| 3 | 3 | $11$ | $2 \cdot 1 + 1 = 3$ |
| 4 | 1 | $100$ | $2 \cdot 2 - 1$... 아니, $2^2 + 0$, $2 \cdot 0 + 1 = 1$ |
| 5 | 3 | $101$ | $2^2 + 1$, $2 \cdot 1 + 1 = 3$ |
| 6 | 5 | $110$ | $2^2 + 2$, $2 \cdot 2 + 1 = 5$ |
| 7 | 7 | $111$ | $2^2 + 3$, $2 \cdot 3 + 1 = 7$ |
| 8 | 1 | $1000$ | $2^3 + 0$, $2 \cdot 0 + 1 = 1$ |

흥미로운 패턴이 있다. $n$의 이진 표현에서 **최상위 비트를 맨 뒤로 옮기면** $J(n)$이 된다.

- $n = 6 = (110)_2$ → $(101)_2 = 5 = J(6)$  
- $n = 7 = (111)_2$ → $(111)_2 = 7 = J(7)$  
- $n = 12 = (1100)_2$ → $(1001)_2 = 9 = J(12)$  

### Decrease-by-Half 관점

점화식 $J(2n) = 2J(n) - 1$에서 볼 수 있듯이, $n$명의 요세푸스 문제를 $\lfloor n/2 \rfloor$명의 문제로 환원한다. 전형적인 Decrease-by-Constant-Factor 구조다.

### C++ 구현

```cpp
// 재귀 방식
int josephus(int n) {
    if (n == 1) return 1;
    if (n % 2 == 0)
        return 2 * josephus(n / 2) - 1;
    else
        return 2 * josephus(n / 2) + 1;
}

// 비트 연산 방식 (최상위 비트를 맨 뒤로 이동)
int josephusBit(int n) {
    // 최상위 비트의 위치를 구함
    int highestBit = 1;
    while (highestBit <= n) highestBit <<= 1;
    highestBit >>= 1;

    // 최상위 비트를 제거하고 맨 뒤에 1을 붙임
    return ((n ^ highestBit) << 1) | 1;
}
```

---

## 네 알고리즘 비교 요약

| 알고리즘 | 문제 유형 | 감소 방식 | 단계당 연산 | 최종 복잡도 |
|---|---|---|---|---|
| Binary Search | 탐색 | $n \to \lfloor n/2 \rfloor$ | 비교 1회 | $\Theta(\log n)$ |
| Fake-Coin | 탐색 | $n \to \lfloor n/2 \rfloor$ | 저울질 1회 | $\Theta(\log n)$ |
| Russian Peasant | 계산 | $n \to \lfloor n/2 \rfloor$ | 조건 검사 + 덧셈 | $\Theta(\log n)$ |
| Josephus | 계산 | $n \to \lfloor n/2 \rfloor$ | 산술 연산 | $\Theta(\log n)$ |

네 알고리즘 모두 매 단계마다 문제 크기를 절반으로 줄이므로 $\Theta(\log n)$의 복잡도를 달성한다. 이것이 Decrease-by-Constant-Factor의 핵심적인 장점이다.

