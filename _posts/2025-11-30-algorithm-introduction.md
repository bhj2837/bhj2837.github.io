---
title: "알고리즘  - 기본 정의와 개념"
date: 2025-11-30 16:00:00 +0900
categories: [Algorithm Basics, Introduction]
tags: [algorithm, complexity, big-o, time-complexity, space-complexity]
math: true
---

## 알고리즘이란?

**알고리즘(Algorithm)**은 어떤 문제를 해결하기 위한 명확하고 유한한 단계의 절차이며, 입력을 받아서 원하는 출력을 만들어내는 일련의 계산 과정이라고 할 수 있다.

### 알고리즘의 조건

좋은 알고리즘이 되기 위해서는 다음 조건들을 만족해야 한다:

1. **입력(Input)**: 0개 이상의 입력이 존재
2. **출력(Output)**: 1개 이상의 출력이 존재
3. **명확성(Definiteness)**: 각 단계는 명확하고 모호하지 않음
4. **유한성(Finiteness)**: 유한한 단계 후에 반드시 종료
5. **효율성(Effectiveness)**: 각 단계는 실행 가능

## 알고리즘의 필요성

같은 문제를 해결하는 여러 알고리즘이 존재할 수 있으나 이들의 **효율성**은 천차만별이다. 

예를 들어, 정렬되지 않은 배열에서 특정 값을 찾는 문제를 고려할 때:
- **방법 1**: 처음부터 끝까지 하나씩 확인 → 최악의 경우 n번 확인
- **방법 2**: 먼저 정렬한 후 이진 탐색 → 훨씬 빠른 탐색 가능

데이터가 작을 때는 차이가 미미하지만, 데이터가 커질수록 성능 차이가 기하급수적으로 벌어진다. 

## 알고리즘의 성능 분석

알고리즘의 효율성을 평가하는 두 가지 주요 척도가 있다:

### 1. 시간 복잡도 (Time Complexity)

**시간 복잡도**는 알고리즘이 실행되는데 걸리는 시간을 입력 크기의 함수로 나타낸 것이다.

#### Big-O 표기법

Big-O 표기법은 알고리즘의 **최악의 경우** 성능을 나타내며. 입력 크기 n이 커질 때 실행 시간이 어떻게 증가하는지를 보여준다.

**주요 시간 복잡도 (빠른 순서):**

| 표기법 | 이름 | 예시 |
|--------|------|------|
| O(1) | 상수 시간 | 배열의 인덱스 접근 |
| O(log n) | 로그 시간 | 이진 탐색 |
| O(n) | 선형 시간 | 배열 순회 |
| O(n log n) | 선형 로그 시간 | 병합 정렬, 퀵 정렬 |
| O(n²) | 이차 시간 | 버블 정렬, 선택 정렬 |
| O(2ⁿ) | 지수 시간 | 피보나치 수열 (재귀) |
| O(n!) | 팩토리얼 시간 | 순열 생성 |

#### 예시: 배열 합 구하기

```cpp
int sumArray(vector<int>& arr) {
    int total = 0;
    for (int num : arr) {
        total += num;
    }
    return total;
}
```

이 알고리즘의 시간 복잡도는 **O(n)**이다.
- 배열의 모든 요소를 한 번씩 방문하므로
- n개의 요소가 있다면 n번 반복

#### 예시: 이중 반복문

```cpp
void printPairs(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            cout << arr[i] << " " << arr[j] << endl;
        }
    }
}
```

이 알고리즘의 시간 복잡도는 **O(n²)**이다.
- 외부 반복문이 n번 실행
- 각 외부 반복마다 내부 반복문이 n번 실행
- 총 n × n = n² 번 실행

### 2. 공간 복잡도 (Space Complexity)

**공간 복잡도**는 알고리즘이 실행되는데 필요한 메모리 공간을 입력 크기의 함수로 나타낸 것이다.

#### 공간 복잡도의 구성 요소

1. **고정 공간**: 입력 크기와 무관하게 필요한 공간 (코드, 상수, 변수 등)
2. **가변 공간**: 입력 크기에 따라 달라지는 공간

#### 예시: In-place 알고리즘

```cpp
void reverseArray(vector<int>& arr) {
    int left = 0;
    int right = arr.size() - 1;
    while (left < right) {
        swap(arr[left], arr[right]);
        left++;
        right--;
    }
}
```

이 알고리즘의 공간 복잡도는 **O(1)**이다.
- 추가 배열을 생성하지 않음
- left, right 변수만 사용 (상수 공간)

#### 예시: 추가 배열 사용

```cpp
vector<int> createDoubledArray(vector<int>& arr) {
    vector<int> result;
    for (int num : arr) {
        result.push_back(num * 2);
    }
    return result;
}
```

이 알고리즘의 공간 복잡도는 **O(n)**이다.
- 입력 배열과 같은 크기의 result 배열 생성
- n개의 요소를 저장하므로 O(n)

## 시간 복잡도 vs 공간 복잡도 트레이드오프

일반적인 경우, 시간 복잡도와 공간 복잡도 사이에 **트레이드오프(trade-off)**, 즉 시간을 줄이려면 메모리를 더 사용하고, 메모리를 줄이려면 시간이 걸리는 상충 관계가 형성된다.
이 현상은 더 빠르게 계산하려면 중간 결과를 저장해야 하고, 저장 공간을 아끼려면 매번 다시 계산해야 하기 때문에 발생한다.

### 예시: 피보나치 수열

**방법 1: 단순 재귀 (시간 느림, 공간 적음)**
```cpp
int fibonacci(int n) {
    if (n <= 1) return n;
    return fibonacci(n-1) + fibonacci(n-2);
}
```
- 시간 복잡도: O(2ⁿ) - 매우 느림
- 공간 복잡도: O(n) - 재귀 호출 스택

**방법 2: 동적 프로그래밍 (시간 빠름, 공간 많음)**
```cpp
int fibonacci(int n) {
    if (n <= 1) return n;
    vector<int> dp(n + 1);
    dp[0] = 0;
    dp[1] = 1;
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i-1] + dp[i-2];
    }
    return dp[n];
}
```
- 시간 복잡도: O(n) - 빠름
- 공간 복잡도: O(n) - 배열 저장

**방법 3: 최적화된 동적 프로그래밍 (둘 다 효율적)**
```cpp
int fibonacci(int n) {
    if (n <= 1) return n;
    int prev = 0, curr = 1;
    for (int i = 2; i <= n; i++) {
        int next = prev + curr;
        prev = curr;
        curr = next;
    }
    return curr;
}
```
- 시간 복잡도: O(n) - 빠름
- 공간 복잡도: O(1) - 변수 2개만 사용

## Big-O 표기법의 특징

### 1. 최고차항만 고려

$$O(3n^2 + 5n + 10) = O(n^2)$$

n이 충분히 크면 최고차항이 지배적이므로, 하위 차수와 상수는 무시한다.

### 2. 상수 계수 무시

$$O(5n) = O(n)$$

Big-O는 증가율을 나타내므로 상수 배수는 중요하지 않다.

### 3. 최악의 경우 분석

Big-O는 일반적으로 **최악의 경우(Worst Case)**를 나타낸다.

다른 표기법:
- **Ω (Omega)**: 최선의 경우 (Best Case)
- **Θ (Theta)**: 평균의 경우 (Average Case)

## 예제: 두 수의 합 찾기

문제: 정렬되지 않은 배열에서 합이 target이 되는 두 수의 인덱스를 찾으시오.

### 방법 1: 브루트 포스 (Brute Force)

```cpp
vector<int> twoSumBrute(vector<int>& nums, int target) {
    int n = nums.size();
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            if (nums[i] + nums[j] == target) {
                return {i, j};
            }
        }
    }
    return {};
}
```

- 시간 복잡도: **O(n²)** - 이중 반복문
- 공간 복잡도: **O(1)** - 추가 공간 없음

### 방법 2: 해시 테이블

```cpp
vector<int> twoSumHash(vector<int>& nums, int target) {
    unordered_map<int, int> hashMap;
    for (int i = 0; i < nums.size(); i++) {
        int complement = target - nums[i];
        if (hashMap.find(complement) != hashMap.end()) {
            return {hashMap[complement], i};
        }
        hashMap[nums[i]] = i;
    }
    return {};
}
```

- 시간 복잡도: **O(n)** - 배열 한 번 순회
- 공간 복잡도: **O(n)** - 해시 테이블 사용

해시 테이블을 사용하면 시간을 대폭 줄일 수 있지만, 추가 메모리가 필요하다.




**다음 글**: [점진적 접근법과 정렬 알고리즘](#)