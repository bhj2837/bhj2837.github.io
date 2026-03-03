---
title: "Divide-and-Conquer (2): 큰 정수 곱셈, Strassen 행렬 곱셈, 최근접 쌍, 볼록 껍질"
date: 2026-03-03 18:00:00 +0900
categories: [Problem Solving Strategies, Divide and Conquer]
tags: [algorithm, divide-and-conquer, strassen, closest-pair, convex-hull, large-integer]
math: true
---



## 1. 큰 정수 곱셈

### 문제 정의

컴퓨터의 기본 자료형(int, long long)은 수십 자리의 정수를 직접 다룰 수 없다. 암호학 등의 분야에서는 수백 자리 이상의 정수 곱셈이 필요하다. 이를 어떻게 효율적으로 처리할 수 있을까?

### 브루트 포스 방식

우리가 손으로 하는 곱셈과 같다. $n$자리 정수 두 개를 곱할 때 각 자릿수 쌍마다 곱셈을 수행한다.

$$\underbrace{a_1 a_2 \cdots a_n}_{n\text{자리}} \times \underbrace{b_1 b_2 \cdots b_n}_{n\text{자리}} \Rightarrow \Theta(n^2) \text{ 단일 자릿수 곱셈}$$

### Divide-and-Conquer 방식

$n$자리 정수 $X$, $Y$를 각각 **절반씩 분할**한다. $n$이 짝수라고 가정하면:

$$X = A \cdot 10^{n/2} + B, \quad Y = C \cdot 10^{n/2} + D$$

여기서 $A$, $B$, $C$, $D$는 모두 $n/2$자리 정수다. 그러면:

$$X \cdot Y = (A \cdot 10^{n/2} + B)(C \cdot 10^{n/2} + D)$$

$$= AC \cdot 10^{n} + (AD + BC) \cdot 10^{n/2} + BD$$

이 전개식에는 $AC$, $AD$, $BC$, $BD$ 총 **4번의 $n/2$자리 곱셈**이 필요하다.

### 점화식 및 복잡도

4번의 재귀 곱셈이 필요할 경우 점화식은 다음과 같다.

$$M(n) = 4M(n/2), \quad M(1) = 1$$

Master Theorem을 적용하면 $a=4$, $b=2$, $d=0$이므로 $a > b^d$:

$$M(n) \in \Theta(n^{\log_2 4}) = \Theta(n^2)$$

브루트 포스와 동일하다. 분할만으로는 개선이 없다.

### Karatsuba의 아이디어: 곱셈 3번으로 줄이기

$AD + BC$ 항을 영리하게 계산하면 곱셈을 **3번**으로 줄일 수 있다.

다음 세 가지 값을 정의한다.

$$r = AC, \quad s = BD, \quad t = (A+B)(C+D) = AC + AD + BC + BD$$

그러면 우리가 원하는 중간 항은:

$$AD + BC = t - r - s$$

즉, 추가적인 곱셈 없이 덧셈/뺄셈만으로 $AD + BC$를 구할 수 있다. 최종 결과:

$$X \cdot Y = r \cdot 10^{n} + (t - r - s) \cdot 10^{n/2} + s$$

이제 점화식이 바뀐다.

$$M(n) = 3M(n/2), \quad M(1) = 1$$

$a=3$, $b=2$, $d=0$이므로:

$$M(n) \in \Theta(n^{\log_2 3}) \approx \Theta(n^{1.585})$$

### 브루트 포스 vs Divide-and-Conquer 비교

| | 브루트 포스 | D&C (4회 곱셈) | D&C (3회, Karatsuba) |
|---|:---:|:---:|:---:|
| 시간 복잡도 | $\Theta(n^2)$ | $\Theta(n^2)$ | $\Theta(n^{1.585})$ |
| $n=100$자리 | $10,000$ | $10,000$ | $\approx 3,162$ |
| $n=1,000$자리 | $1,000,000$ | $1,000,000$ | $\approx 316,228$ |
| $n=10,000$자리 | $10^8$ | $10^8$ | $\approx 10^{6.3}$ |

실제로 이 Divide-and-Conquer 알고리즘은 **8자리 정수부터** 브루트 포스를 앞지르고, **300자리 이상**에서는 2배 이상 빠르다고 보고되어 있다. 현대 암호학에서 특히 중요한 영역이다.

### C++ 구현 (개념적 구조)

```cpp
#include <iostream>
#include <string>
#include <cmath>
using namespace std;

// 큰 정수를 long long으로 표현하는 단순 구현 (개념 전달용)
long long karatsubaSimple(long long x, long long y) {
    // base case: 한 자리 수
    if (x < 10 || y < 10) return x * y;

    int n = to_string(max(x, y)).size();
    int half = n / 2;
    long long p = (long long)pow(10, half);

    long long a = x / p, b = x % p;
    long long c = y / p, d = y % p;

    long long r = karatsubaSimple(a, c);          // AC
    long long s = karatsubaSimple(b, d);          // BD
    long long t = karatsubaSimple(a + b, c + d);  // (A+B)(C+D)

    return r * p * p + (t - r - s) * p + s;
}

int main() {
    cout << karatsubaSimple(1234, 5678) << "\n"; // 7006652
    return 0;
}
```

---

## 2. Strassen 행렬 곱셈

### 브루트 포스 방식

$n \times n$ 행렬 곱셈 $C = A \times B$는 다음과 같이 계산된다.

$$C[i][j] = \sum_{k=1}^{n} A[i][k] \cdot B[k][j]$$

$n \times n$ 행렬 곱셈에 필요한 연산 수:

| 연산 | 횟수 |
|---|:---:|
| 곱셈 | $n^3$ |
| 덧셈 | $n^3 - n^2$ |
| **시간 복잡도** | $\Theta(n^3)$ |

### Divide-and-Conquer 기본 접근

$n$이 2의 거듭제곱일 때, $n \times n$ 행렬을 $n/2 \times n/2$ 블록 행렬 4개로 분할한다.

$$A = \begin{pmatrix} A_{00} & A_{01} \\ A_{10} & A_{11} \end{pmatrix}, \quad B = \begin{pmatrix} B_{00} & B_{01} \\ B_{10} & B_{11} \end{pmatrix}, \quad C = \begin{pmatrix} C_{00} & C_{01} \\ C_{10} & C_{11} \end{pmatrix}$$

블록 행렬 곱셈:

$$C_{00} = A_{00}B_{00} + A_{01}B_{10}, \quad C_{01} = A_{00}B_{01} + A_{01}B_{11}$$

$$C_{10} = A_{10}B_{00} + A_{11}B_{10}, \quad C_{11} = A_{10}B_{01} + A_{11}B_{11}$$

이 방식은 $n/2 \times n/2$ 행렬 곱셈 **8번**과 덧셈 4번이 필요하다.

$$M(n) = 8M(n/2) \Rightarrow \Theta(n^{\log_2 8}) = \Theta(n^3)$$

역시 브루트 포스와 동일하다.

### Strassen의 아이디어: 곱셈 7번으로 줄이기

Volker Strassen(1969)은 8번의 행렬 곱셈을 **7번**으로 줄이는 방법을 발견했다. 다음 7개의 보조 행렬을 정의한다.

$$M_1 = (A_{00} + A_{11})(B_{00} + B_{11})$$

$$M_2 = (A_{10} + A_{11})B_{00}$$

$$M_3 = A_{00}(B_{01} - B_{11})$$

$$M_4 = A_{11}(B_{10} - B_{00})$$

$$M_5 = (A_{00} + A_{01})B_{11}$$

$$M_6 = (A_{10} - A_{00})(B_{00} + B_{01})$$

$$M_7 = (A_{01} - A_{11})(B_{10} + B_{11})$$

결과 행렬의 각 블록을 $M_1 \sim M_7$의 조합으로 표현한다.

$$C_{00} = M_1 + M_4 - M_5 + M_7$$

$$C_{01} = M_3 + M_5$$

$$C_{10} = M_2 + M_4$$

$$C_{11} = M_1 - M_2 + M_3 + M_6$$

곱셈 횟수는 7번, 덧셈/뺄셈은 18번이다.

### 복잡도 분석

$$M(n) = 7M(n/2), \quad M(1) = 1$$

$a=7$, $b=2$, $d=0$이므로 $a > b^d$:

$$M(n) \in \Theta(n^{\log_2 7}) \approx \Theta(n^{2.807})$$

### 브루트 포스 vs Strassen 비교

| | 브루트 포스 | Strassen |
|---|:---:|:---:|
| 재귀 곱셈 횟수 | $8$회 | $7$회 |
| 덧셈/뺄셈 횟수 (블록 기준) | $4$회 | $18$회 |
| 시간 복잡도 | $\Theta(n^3)$ | $\Theta(n^{2.807})$ |
| $n=1,000$ (상대적 연산량) | $10^9$ | $\approx 1.7 \times 10^8$ |
| $n=10,000$ (상대적 연산량) | $10^{12}$ | $\approx 6.4 \times 10^{10}$ |

지수가 $3 \to 2.807$로 줄어든 것이 미미해 보이지만, $n$이 커질수록 격차가 폭발적으로 벌어진다.

### C++ 구현

```cpp
#include <iostream>
#include <vector>
using namespace std;

using Matrix = vector<vector<long long>>;

Matrix add(const Matrix& A, const Matrix& B) {
    int n = A.size();
    Matrix C(n, vector<long long>(n, 0));
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            C[i][j] = A[i][j] + B[i][j];
    return C;
}

Matrix sub(const Matrix& A, const Matrix& B) {
    int n = A.size();
    Matrix C(n, vector<long long>(n, 0));
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            C[i][j] = A[i][j] - B[i][j];
    return C;
}

Matrix strassen(const Matrix& A, const Matrix& B) {
    int n = A.size();
    if (n == 1) return {{A[0][0] * B[0][0]}};

    int h = n / 2;
    auto split = [&](const Matrix& M, int r, int c) {
        Matrix S(h, vector<long long>(h));
        for (int i = 0; i < h; i++)
            for (int j = 0; j < h; j++)
                S[i][j] = M[r+i][c+j];
        return S;
    };

    Matrix A00 = split(A,0,0), A01 = split(A,0,h);
    Matrix A10 = split(A,h,0), A11 = split(A,h,h);
    Matrix B00 = split(B,0,0), B01 = split(B,0,h);
    Matrix B10 = split(B,h,0), B11 = split(B,h,h);

    // 7번의 재귀 곱셈
    Matrix M1 = strassen(add(A00,A11), add(B00,B11));
    Matrix M2 = strassen(add(A10,A11), B00);
    Matrix M3 = strassen(A00,           sub(B01,B11));
    Matrix M4 = strassen(A11,           sub(B10,B00));
    Matrix M5 = strassen(add(A00,A01), B11);
    Matrix M6 = strassen(sub(A10,A00), add(B00,B01));
    Matrix M7 = strassen(sub(A01,A11), add(B10,B11));

    Matrix C00 = add(sub(add(M1,M4),M5),M7);
    Matrix C01 = add(M3,M5);
    Matrix C10 = add(M2,M4);
    Matrix C11 = add(sub(add(M1,M3),M2),M6);

    // 결과 합치기
    Matrix C(n, vector<long long>(n));
    for (int i = 0; i < h; i++)
        for (int j = 0; j < h; j++) {
            C[i][j]       = C00[i][j]; C[i][j+h]   = C01[i][j];
            C[i+h][j]     = C10[i][j]; C[i+h][j+h] = C11[i][j];
        }
    return C;
}

int main() {
    // 단위행렬 곱셈 테스트
    Matrix A = {{1,2,3,4},{5,6,7,8},{9,10,11,12},{13,14,15,16}};
    Matrix I = {{1,0,0,0},{0,1,0,0},{0,0,1,0},{0,0,0,1}};
    Matrix C = strassen(A, I);
    for (auto& row : C) {
        for (auto x : row) cout << x << " ";
        cout << "\n";
    }
    return 0;
}
```

---

## 3. 최근접 쌍 문제 (Closest-Pair Problem)

### 문제 정의

평면 위의 $n$개의 점이 주어졌을 때, **가장 가까운 두 점의 쌍**을 찾는 문제다. 브루트 포스로는 모든 쌍을 검사하므로 $\Theta(n^2)$이 걸린다.

### Divide-and-Conquer 전략

1. **Divide**: 점들을 $x$좌표 기준으로 정렬한 후, 수직선 $x = m$을 기준으로 왼쪽 절반과 오른쪽 절반으로 분할한다.
2. **Conquer**: 왼쪽, 오른쪽 각각에서 최근접 쌍 거리를 재귀적으로 구한다.

$$d_L = \text{ClosestPair}(\text{left}), \quad d_R = \text{ClosestPair}(\text{right}), \quad d = \min(d_L, d_R)$$

3. **Combine**: 분할선을 경계로 **넘나드는 쌍** 중 $d$보다 짧은 것이 있는지 확인한다.

### Strip 처리 (핵심 아이디어)

결합 단계에서 경계를 넘나드는 쌍만 후보가 된다. 분할선 $x=m$으로부터 거리 $d$ 이내에 있는 점들만 확인하면 된다.

$$\text{strip} = \{\ p \mid |p.x - m| < d\ \}$$

이 strip 안의 점들을 **$y$좌표 기준으로 정렬**하면, 임의의 점 $p$에 대해 비교해야 할 점은 $y$좌표 차이가 $d$ 미만인 점들뿐이다.

$$|p.y - q.y| < d \text{ 인 } q \text{ 만 비교}$$

기하학적으로, $p$를 기준으로 $2d \times d$ 직사각형 안에 들어갈 수 있는 점의 수는 **최대 8개**로 제한된다. 각 절반의 $d \times d$ 정사각형 안에는 거리 $d$ 이상인 점들만 있으므로 최대 4개만 들어올 수 있기 때문이다.

따라서 결합 단계의 비교는 $O(n)$이다.

### 전체 알고리즘 흐름

| 단계 | 작업 | 복잡도 |
|---|---|:---:|
| 전처리 | $x$좌표 기준 정렬 | $O(n \log n)$ |
| Divide | 중간 인덱스 기준 분할 | $O(1)$ |
| Conquer | 왼쪽/오른쪽 재귀 | $2T(n/2)$ |
| Combine | strip 구성 + $y$ 정렬 + 비교 | $O(n)$ |

### 복잡도 분석

$$T(n) = 2T(n/2) + O(n)$$

$a=2$, $b=2$, $d=1$이므로 $a = b^d$:

$$T(n) \in \Theta(n \log n)$$

| 방법 | 시간 복잡도 | $n=10^6$일 때 (상대적) |
|---|:---:|:---:|
| 브루트 포스 | $\Theta(n^2)$ | $10^{12}$ |
| Divide-and-Conquer | $\Theta(n \log n)$ | $\approx 2 \times 10^7$ |

### C++ 구현

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath>
#include <cfloat>
using namespace std;

struct Point { double x, y; };

double dist(const Point& a, const Point& b) {
    return sqrt((a.x-b.x)*(a.x-b.x) + (a.y-b.y)*(a.y-b.y));
}

double bruteForce(vector<Point>& pts, int l, int r) {
    double d = DBL_MAX;
    for (int i = l; i <= r; i++)
        for (int j = i+1; j <= r; j++)
            d = min(d, dist(pts[i], pts[j]));
    return d;
}

double stripClosest(vector<Point> strip, double d) {
    double minD = d;
    sort(strip.begin(), strip.end(), [](const Point& a, const Point& b){
        return a.y < b.y;
    });
    for (int i = 0; i < (int)strip.size(); i++)
        for (int j = i+1;
             j < (int)strip.size() && (strip[j].y - strip[i].y) < minD; j++)
            minD = min(minD, dist(strip[i], strip[j]));
    return minD;
}

double closestPairRec(vector<Point>& pts, int l, int r) {
    if (r - l <= 2) return bruteForce(pts, l, r);

    int mid = (l + r) / 2;
    double mx = pts[mid].x;

    double d = min(closestPairRec(pts, l, mid),
                   closestPairRec(pts, mid+1, r));

    // strip 구성: 분할선으로부터 d 이내의 점
    vector<Point> strip;
    for (int i = l; i <= r; i++)
        if (abs(pts[i].x - mx) < d)
            strip.push_back(pts[i]);

    return min(d, stripClosest(strip, d));
}

double closestPair(vector<Point>& pts) {
    sort(pts.begin(), pts.end(), [](const Point& a, const Point& b){
        return a.x < b.x;
    });
    return closestPairRec(pts, 0, (int)pts.size()-1);
}

int main() {
    vector<Point> pts = {{2,3},{12,30},{40,50},{5,1},{12,10},{3,4}};
    cout << "최근접 거리: " << closestPair(pts) << "\n"; // 약 1.414
    return 0;
}
```

---

## 4. 볼록 껍질 문제 (Convex-Hull Problem)

### 문제 정의

평면 위의 $n$개의 점을 모두 포함하는 **가장 작은 볼록 다각형**을 구하는 문제다. 브루트 포스 알고리즘은 두 점이 이루는 선분을 검사해 나머지 점이 모두 같은 방향에 있는지 확인하므로 $\Theta(n^3)$이 걸린다.

![Convex Hull 예시](/assets/img/posts/convex-hull-example.png)
_왼쪽: 입력 점들 / 오른쪽: 빨간 점(껍질 꼭짓점)을 연결한 볼록 껍질과 내부 점(파란 점)_

### Divide-and-Conquer 전략

1. **Divide**: 점들을 $x$좌표 기준으로 정렬한 후, 왼쪽 절반과 오른쪽 절반으로 분할한다.
2. **Conquer**: 왼쪽 볼록 껍질 $\text{CH}(L)$과 오른쪽 볼록 껍질 $\text{CH}(R)$을 각각 재귀적으로 구한다.
3. **Combine**: 두 볼록 껍질을 **공통 접선(common tangent)**으로 연결하여 병합한다.

### Combine 단계: 공통 접선 찾기

두 볼록 껍질 $\text{CH}(L)$과 $\text{CH}(R)$을 합치려면 **위쪽 공통 접선**과 **아래쪽 공통 접선**을 찾아야 한다.

| 접선 종류 | 조건 |
|---|---|
| **Upper tangent** | 두 껍질의 모든 점이 접선 **아래**에 있어야 한다 |
| **Lower tangent** | 두 껍질의 모든 점이 접선 **위**에 있어야 한다 |

두 접선이 결정되면, 각 껍질에서 접선 사이에 해당하는 내부 점들을 제거해 합쳐진 볼록 껍질을 완성한다.

### 복잡도 분석

$$T(n) = 2T(n/2) + O(n)$$

Combine 단계에서 공통 접선 탐색이 $O(n)$이므로:

$$T(n) \in \Theta(n \log n)$$

| 방법 | 시간 복잡도 |
|---|:---:|
| 브루트 포스 | $\Theta(n^3)$ |
| Divide-and-Conquer | $\Theta(n \log n)$ |
| Graham Scan | $\Theta(n \log n)$ |

### C++ 구현 (Andrew's Monotone Chain)

Divide-and-Conquer의 결합 단계와 동등한 결과를 효율적으로 산출하는 구현이다.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

struct Point {
    long long x, y;
    bool operator<(const Point& o) const {
        return x < o.x || (x == o.x && y < o.y);
    }
};

// 외적: 양수=반시계, 음수=시계, 0=일직선
long long cross(const Point& O, const Point& A, const Point& B) {
    return (A.x - O.x) * (B.y - O.y) - (A.y - O.y) * (B.x - O.x);
}

vector<Point> convexHull(vector<Point> pts) {
    int n = pts.size(), k = 0;
    if (n < 3) return pts;

    sort(pts.begin(), pts.end());
    vector<Point> hull(2 * n);

    // 하단 껍질 (왼쪽 → 오른쪽)
    for (int i = 0; i < n; i++) {
        while (k >= 2 && cross(hull[k-2], hull[k-1], pts[i]) <= 0) k--;
        hull[k++] = pts[i];
    }
    // 상단 껍질 (오른쪽 → 왼쪽)
    for (int i = n-2, t = k+1; i >= 0; i--) {
        while (k >= t && cross(hull[k-2], hull[k-1], pts[i]) <= 0) k--;
        hull[k++] = pts[i];
    }

    hull.resize(k - 1);
    return hull;
}

int main() {
    vector<Point> pts = {{0,3},{1,1},{2,2},{4,4},{0,0},{1,2},{3,1},{3,3}};
    vector<Point> hull = convexHull(pts);

    cout << "볼록 껍질 꼭짓점 (" << hull.size() << "개):\n";
    for (auto& p : hull)
        cout << "(" << p.x << ", " << p.y << ")\n";
    return 0;
}
```

---


