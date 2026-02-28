---
title: "Divide-and-Conquer (1): 개념, Mergesort, Quicksort, 이진 트리 순회"
date: 2026-02-28 18:00:00 +0900
categories: [Problem Solving Strategies, Divide and Conquer]
tags: [algorithm, divide-and-conquer, mergesort, quicksort, binary-tree, traversal]
math: true
---

## 1. Divide-and-Conquer란?

Divide-and-Conquer는 다음 세 단계로 이루어진다.

1. **Divide(분할)**: 주어진 문제를 같은 유형의 더 작은 부분 문제 여러 개로 나눈다.
2. **Conquer(정복)**: 각 부분 문제를 (보통 재귀적으로) 풀이한다.
3. **Combine(결합)**: 부분 문제들의 해를 합쳐 원래 문제의 해를 구한다.

### Decrease-and-Conquer와의 차이

| | Decrease-and-Conquer | Divide-and-Conquer |
|---|---|---|
| 부분 문제 수 | **1개** | **여러 개** |
| 결합 단계 | 불필요 (또는 단순) | 필요 |
| 대표 예시 | 삽입 정렬, 이진 탐색 | Mergesort, Quicksort |

Decrease-and-Conquer는 문제를 하나씩 줄여나가지만, Divide-and-Conquer는 문제를 동시에 여러 개로 쪼갠다. 이 구조 덕분에 각 부분 문제를 **별도의 프로세서에서 병렬로 풀 수 있어** 병렬 계산에 특히 적합하다.

### 대표적인 응용 알고리즘

- 정렬: Mergesort, Quicksort
- 이진 트리 순회 (Preorder, Inorder, Postorder)
- 큰 정수 곱셈
- Strassen 행렬 곱셈
- 최근접 쌍 문제, 볼록 껍질 문제

이번 포스트에서는 이 중 **Mergesort, Quicksort, 이진 트리 순회**를 다룬다.

---

## 2. Mergesort

### 동작 원리

Mergesort는 Divide-and-Conquer의 가장 교과서적인 예시다.

1. 배열을 **절반으로 분할**한다.
2. 왼쪽 절반과 오른쪽 절반을 **각각 재귀적으로 정렬**한다.
3. 두 정렬된 배열을 **병합(merge)**하여 하나의 정렬된 배열을 만든다.

핵심은 3단계 **병합** 과정이다. Two-pointer 기법을 사용한다.

- 두 배열의 시작 위치를 가리키는 포인터(인덱스)를 초기화한다.
- 두 포인터가 가리키는 원소를 비교해 **더 작은 원소**를 새 배열에 추가하고, 해당 포인터를 한 칸 전진시킨다.
- 어느 한 배열이 소진될 때까지 반복한 뒤, 남은 배열의 원소를 전부 복사한다.

### C++ 구현

```cpp
#include <iostream>
#include <vector>
using namespace std;

void merge(vector<int>& arr, int left, int mid, int right) {
    vector<int> temp;
    int i = left, j = mid + 1;

    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j])
            temp.push_back(arr[i++]);
        else
            temp.push_back(arr[j++]);
    }
    while (i <= mid)  temp.push_back(arr[i++]);
    while (j <= right) temp.push_back(arr[j++]);

    for (int k = 0; k < (int)temp.size(); k++)
        arr[left + k] = temp[k];
}

void mergesort(vector<int>& arr, int left, int right) {
    if (left >= right) return;

    int mid = (left + right) / 2;
    mergesort(arr, left, mid);
    mergesort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}

int main() {
    vector<int> arr = {38, 27, 43, 3, 9, 82, 10};
    mergesort(arr, 0, arr.size() - 1);

    for (int x : arr) cout << x << " ";
    return 0;
}
```

### 복잡도 분석

Mergesort의 비교 횟수를 $C(n)$이라 하면 다음 점화식이 성립한다.

$$C(n) = 2C(n/2) + C_{\text{merge}}(n), \quad C(1) = 0$$

병합 단계에서 최악의 경우 $n-1$번 비교가 발생하므로 $C_{\text{merge}}(n) \leq n-1$이다. Master Theorem을 적용하면:

$$C(n) \in \Theta(n \log n)$$

최선, 최악, 평균 모두 $\Theta(n \log n)$이다. 단, 병합을 위한 **추가 메모리 $O(n)$**이 필요하다는 단점이 있다.

---

## 3. Quicksort

### Mergesort와의 비교

Mergesort와 Quicksort는 둘 다 Divide-and-Conquer이지만 분할 방식이 근본적으로 다르다.

| | Mergesort | Quicksort |
|---|---|---|
| 분할 기준 | **위치** (배열의 중간) | **값** (피벗 기준) |
| 핵심 작업 위치 | Combine 단계 (병합) | Divide 단계 (파티션) |
| 추가 메모리 | $O(n)$ | $O(\log n)$ (재귀 스택) |

Quicksort는 **파티션(partition)** 과정에서 모든 핵심 작업이 이루어진다. 파티션이 완료되면 피벗 원소 `A[s]`는 이미 최종 위치에 놓이고, 피벗 왼쪽은 모두 `A[s]` 이하, 오른쪽은 모두 `A[s]` 이상이 된다. 이후 두 부분을 독립적으로 재귀 정렬하면 된다.

### Hoare의 파티셔닝 알고리즘

Quicksort를 고안한 C.A.R. Hoare가 제안한 파티셔닝 방법이다. 배열의 첫 원소를 피벗 $p$로 선택하고 양쪽 끝에서 스캔을 진행한다.

1. **왼쪽 스캔** (인덱스 $i$): 피벗보다 **크거나 같은** 원소를 만나면 멈춘다.
2. **오른쪽 스캔** (인덱스 $j$): 피벗보다 **작거나 같은** 원소를 만나면 멈춘다.
3. 두 스캔이 멈춘 후 세 가지 상황이 발생한다.
   - $i < j$ (교차 전): `A[i]`와 `A[j]`를 교환하고 스캔을 재개한다.
   - $i > j$ (교차 후): 파티션 완료. 피벗 `A[0]`과 `A[j]`를 교환한다.
   - $i = j$ (동일 위치): 해당 값이 피벗과 같으므로 $i \geq j$ 조건으로 위 두 경우를 통합 처리한다.

파티션 완료 후 `A[j]`가 피벗의 최종 위치가 된다.

#### 파티셔닝 예시

배열 `[5, 3, 8, 4, 2]`, 피벗 $p = 5$:

| 단계 | 배열 상태 | 설명 |
|---|---|---|
| 초기 | `[5, 3, 8, 4, 2]` | $i=1$, $j=4$ |
| 스캔 | $i$: 8에서 멈춤, $j$: 2에서 멈춤 | $i < j$이므로 교환 |
| 교환 후 | `[5, 3, 2, 4, 8]` | 스캔 재개 |
| 스캔 | $i$: 8에서 멈춤, $j$: 4에서 멈춤 | $i > j$이므로 파티션 완료 |
| 피벗 교환 | `[4, 3, 2, 5, 8]` | 피벗 5가 인덱스 3에 안착 |

### C++ 구현

```cpp
#include <iostream>
#include <vector>
using namespace std;

int partition(vector<int>& arr, int left, int right) {
    int pivot = arr[left];
    int i = left + 1, j = right;

    while (true) {
        while (i <= right && arr[i] < pivot) i++;
        while (j > left  && arr[j] > pivot) j--;
        if (i >= j) break;
        swap(arr[i], arr[j]);
        i++; j--;
    }
    swap(arr[left], arr[j]);
    return j;
}

void quicksort(vector<int>& arr, int left, int right) {
    if (left >= right) return;

    int s = partition(arr, left, right);
    quicksort(arr, left, s - 1);
    quicksort(arr, s + 1, right);
}

int main() {
    vector<int> arr = {5, 3, 8, 4, 2, 7, 1, 6};
    quicksort(arr, 0, arr.size() - 1);

    for (int x : arr) cout << x << " ";
    return 0;
}
```

### 복잡도 분석

Quicksort의 복잡도는 **파티션이 얼마나 균형 잡히느냐**에 따라 크게 달라진다.

**최선/평균 $\Theta(n \log n)$**: 피벗이 매번 중간값에 가까워 균형 있게 분할될 때

$$C_{\text{best}}(n) = 2C(n/2) + (n-1) \approx n \log_2 n$$

**최악 $\Theta(n^2)$**: 피벗이 항상 최솟값 또는 최댓값으로 선택될 때 (이미 정렬된 배열에 첫 원소를 피벗으로 사용하는 경우)

$$C_{\text{worst}}(n) = (n-1) + (n-2) + \cdots + 1 = \frac{n(n-1)}{2} \in \Theta(n^2)$$

실용적으로는 피벗 선택 전략(랜덤 피벗, 세 값의 중앙값 등)으로 최악 케이스를 피할 수 있어 평균적으로 매우 빠른 알고리즘이다.

---

## 4. 이진 트리 순회

### 이진 트리의 높이

이진 트리의 **높이(height)**는 루트에서 가장 먼 리프까지의 경로 길이로 정의된다. 이를 재귀적으로 계산할 수 있다.

$$h(T) = \max(h(T_L),\ h(T_R)) + 1$$

리프 노드의 높이는 0, 빈 트리의 높이는 $-1$로 정의한다. 이 자체가 Divide-and-Conquer 구조로, 왼쪽 서브트리와 오른쪽 서브트리의 높이를 각각 구한 뒤 결합한다.

### 세 가지 순회 방식

이진 트리의 가장 중요한 Divide-and-Conquer 알고리즘은 세 가지 고전적 순회다. 루트, 왼쪽 서브트리(L), 오른쪽 서브트리(R)를 방문하는 순서에 따라 구분된다.

| 순회 방식 | 방문 순서 | 특징 |
|---|---|---|
| **Preorder** | 루트 → L → R | 루트를 가장 먼저 방문 |
| **Inorder** | L → 루트 → R | BST에서 오름차순 출력 |
| **Postorder** | L → R → 루트 | 루트를 가장 나중에 방문 |

각 순회에서 세 단계(루트 방문, L 서브트리 순회, R 서브트리 순회)는 모두 재귀적으로 이루어진다. 이것이 바로 Divide-and-Conquer 구조다.

### C++ 구현

```cpp
#include <iostream>
using namespace std;

struct Node {
    int val;
    Node* left;
    Node* right;
    Node(int v) : val(v), left(nullptr), right(nullptr) {}
};

// 트리 높이 계산
int height(Node* root) {
    if (root == nullptr) return -1;
    return max(height(root->left), height(root->right)) + 1;
}

// Preorder: 루트 → L → R
void preorder(Node* root) {
    if (root == nullptr) return;
    cout << root->val << " ";
    preorder(root->left);
    preorder(root->right);
}

// Inorder: L → 루트 → R
void inorder(Node* root) {
    if (root == nullptr) return;
    inorder(root->left);
    cout << root->val << " ";
    inorder(root->right);
}

// Postorder: L → R → 루트
void postorder(Node* root) {
    if (root == nullptr) return;
    postorder(root->left);
    postorder(root->right);
    cout << root->val << " ";
}

int main() {
    // 예시 트리 구성
    //       1
    //      / \
    //     2   3
    //    / \
    //   4   5
    Node* root = new Node(1);
    root->left  = new Node(2);
    root->right = new Node(3);
    root->left->left  = new Node(4);
    root->left->right = new Node(5);

    cout << "높이: "    << height(root)   << "\n";
    cout << "Preorder:  "; preorder(root);  cout << "\n";
    cout << "Inorder:   "; inorder(root);   cout << "\n";
    cout << "Postorder: "; postorder(root); cout << "\n";

    return 0;
}
```

**출력 결과**:

```
높이: 2
Preorder:  1 2 4 5 3
Inorder:   4 2 5 1 3
Postorder: 4 5 2 3 1
```

### Variable-size-decrease와의 연결

세 가지 순회는 항상 **두 서브트리를 모두 방문**하므로 Divide-and-Conquer에 해당한다. 그러나 이진 탐색 트리(BST)에서의 **탐색과 삽입**은 값의 비교 결과에 따라 왼쪽 또는 오른쪽 **하나의 서브트리만** 방문한다. 이는 앞서 살펴본 Variable-size-decrease의 전형적인 예시로, Divide-and-Conquer가 아닌 Decrease-and-Conquer로 분류된다.
