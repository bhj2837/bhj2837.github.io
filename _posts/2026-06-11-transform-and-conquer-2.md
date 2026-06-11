---
title: "Transform-and-Conquer (2): 탐색 트리 — BST, AVL 트리, 2-3 트리"
date: 2026-06-11 18:00:00 +0900
categories: [Problem Solving Strategies, Transform and Conquer]
tags: [algorithm, transform-and-conquer, bst, avl-tree, 2-3-tree, balanced-tree, representation-change]
math: true
---



지난 글에서는 인스턴스 단순화에 속하는 Presorting과 가우스 소거법을 다뤘다. 이번 글은 변환 정복의 또 다른 변형인 표현 변경(representation change)의 대표 사례인 탐색 트리를 살펴본다. 집합을 트리라는 다른 표현으로 바꿔 사전(dictionary) 연산을 효율적으로 처리하는 것이 핵심이다.

## 1. 탐색 알고리즘의 분류

탐색 알고리즘은 크게 세 부류로 나눌 수 있다.

| 부류 | 대표 알고리즘 |
|---|---|
| 리스트 탐색 | 순차 탐색, 이진 탐색, 보간 탐색 |
| 트리 탐색 | 이진 탐색 트리, 균형 이진 트리(AVL·레드블랙), 다진 균형 트리(2-3·2-3-4·B 트리) |
| 해싱 | 개방 해싱(분리 연쇄법), 폐쇄 해싱(개방 주소법) |

이 글에서 다루는 트리 탐색은 사전(삽입·삭제·탐색을 지원하는 자료구조)을 구현하는 핵심 수단이다.

---

## 2. 이진 탐색 트리 (Binary Search Tree)

### 정의

이진 탐색 트리(BST)는 순서를 매길 수 있는 원소들을 노드마다 하나씩 담은 이진 트리로, 모든 노드에서 왼쪽 서브트리의 원소들은 그 노드보다 작고 오른쪽 서브트리의 원소들은 그 노드보다 크다는 성질을 만족한다.

$$\text{left subtree} < K < \text{right subtree}$$

![BST 균형 vs 퇴화](/assets/img/posts/bst-example.png)
_왼쪽: {5,3,8,1,4,7,9}을 넣어 균형 잡힌 BST(높이 2) / 오른쪽: 1~5를 오름차순으로 넣어 사슬로 퇴화한 BST(높이 4)_

집합을 BST로 바꾸는 것 자체가 표현 변경 기법의 한 예다. 정렬되지 않은 원소들의 집합을, 순서 구조를 내장한 트리 표현으로 옮긴 것이다.

### 사전 연산

탐색은 루트에서 시작해 찾는 키와 노드 키를 비교하며 작으면 왼쪽, 크면 오른쪽으로 내려간다. 삽입도 같은 경로를 따라 내려가다 빈 자리에 새 노드를 단다.

```cpp
#include <vector>
using namespace std;

struct Node { int key; Node *left = nullptr, *right = nullptr; };

Node* insert(Node* root, int key) {
    if (!root) return new Node{key};
    if (key < root->key)      root->left  = insert(root->left, key);
    else if (key > root->key) root->right = insert(root->right, key);
    return root; // 중복 키는 무시
}

bool search(Node* root, int key) {
    while (root) {
        if (key == root->key) return true;
        root = (key < root->key) ? root->left : root->right;
    }
    return false;
}
```

### 효율과 한계

탐색·삽입·삭제 모두 루트에서 잎까지 한 경로를 따라가므로, 비용은 트리의 높이에 비례한다. 무작위로 만들어진 트리의 평균 높이는 $\Theta(\log n)$이지만, 정렬된 순서로 원소를 삽입하면 한쪽으로 길게 늘어진 사슬 형태로 퇴화한다.

| 경우 | 트리 높이 | 연산 비용 |
|---|:---:|:---:|
| 평균 (무작위) | $\approx 1.39 \log_2 n$ | $\Theta(\log n)$ |
| 최악 (퇴화) | $n - 1$ | $\Theta(n)$ |

이 최악의 경우를 막는 것이 다음에 볼 균형 탐색 트리의 동기다.

---

## 3. 균형 탐색 트리 (Balanced Search Trees)

BST의 좋은 성질(사전 연산의 로그 효율, 원소가 정렬된 상태로 유지됨)은 살리면서 최악의 퇴화를 피하려는 노력에서 균형 탐색 트리가 나왔다. 접근법은 두 갈래다.

하나는 불균형해진 이진 탐색 트리를 균형 잡힌 트리로 다시 변환하는 방식으로, 인스턴스 단순화에 가깝다. AVL 트리와 레드블랙 트리가 여기에 속한다. 다른 하나는 한 노드에 원소를 두 개 이상 담도록 허용하는 방식으로, 표현 변경에 해당한다. 2-3 트리, 2-3-4 트리, B 트리가 여기에 속한다.

---

## 4. AVL 트리

### 정의

AVL 트리는 모든 노드의 균형 인수(balance factor)가 $-1$, $0$, $+1$ 중 하나인 이진 탐색 트리다. 균형 인수는 그 노드의 왼쪽 서브트리 높이에서 오른쪽 서브트리 높이를 뺀 값으로 정의한다.

$$\text{balance factor}(v) = h(\text{left}(v)) - h(\text{right}(v)) \in \{-1, 0, +1\}$$

### 회전

새 노드를 삽입해 어떤 노드의 균형 인수가 $+2$ 또는 $-2$가 되면, 그 노드를 뿌리로 하는 서브트리에 국소적 변환인 회전(rotation)을 적용해 균형을 되돌린다. 회전은 네 종류다.

| 회전 | 적용 상황 |
|---|---|
| 단일 우회전 (R) | 왼쪽-왼쪽(LL)으로 치우침 |
| 단일 좌회전 (L) | 오른쪽-오른쪽(RR)으로 치우침 |
| 이중 좌우회전 (LR) | 왼쪽-오른쪽으로 치우침 |
| 이중 우좌회전 (RL) | 오른쪽-왼쪽으로 치우침 |

이중 회전은 단일 회전 두 번의 합성이다. 예를 들어 LR 회전은 왼쪽 자식에 좌회전을 적용한 뒤 자신에게 우회전을 적용한 것이다.

![AVL 좌회전](/assets/img/posts/avl-rotation.png)
_$1, 2, 3$을 차례로 삽입하면 루트의 균형 인수가 $-2$가 되어(왼쪽) 오른쪽으로 치우친다. 좌회전(L)을 적용하면 $2$가 루트가 되며 균형이 회복된다(오른쪽)_

### C++ 구현

```cpp
#include <algorithm>
using namespace std;

struct Node {
    int key, height = 1;
    Node *left = nullptr, *right = nullptr;
    Node(int k) : key(k) {}
};

int h(Node* n)  { return n ? n->height : 0; }
int bf(Node* n) { return n ? h(n->left) - h(n->right) : 0; }
void update(Node* n) { n->height = 1 + max(h(n->left), h(n->right)); }

Node* rotateRight(Node* y) {
    Node* x = y->left;  Node* T = x->right;
    x->right = y;       y->left = T;
    update(y);          update(x);
    return x;
}

Node* rotateLeft(Node* x) {
    Node* y = x->right; Node* T = y->left;
    y->left = x;        x->right = T;
    update(x);          update(y);
    return y;
}

Node* insert(Node* node, int key) {
    if (!node) return new Node(key);
    if (key < node->key)      node->left  = insert(node->left, key);
    else if (key > node->key) node->right = insert(node->right, key);
    else return node;
    update(node);

    int balance = bf(node);
    if (balance > 1  && key < node->left->key)  return rotateRight(node);          // LL
    if (balance < -1 && key > node->right->key) return rotateLeft(node);           // RR
    if (balance > 1  && key > node->left->key) {                                   // LR
        node->left = rotateLeft(node->left);  return rotateRight(node);
    }
    if (balance < -1 && key < node->right->key) {                                  // RL
        node->right = rotateRight(node->right); return rotateLeft(node);
    }
    return node;
}
```

$1$부터 $7$까지 오름차순으로 삽입하면, 일반 BST에서는 높이 $7$의 사슬로 퇴화하지만 AVL 트리에서는 회전 덕분에 루트가 $4$, 높이 $3$인 균형 트리가 만들어진다.

### 효율 분석

AVL 트리의 높이는 원소 수 $n$에 대해 다음과 같이 위로 막혀 있다.

$$h \le 1.4405 \log_2 (n + 2) - 1.3277$$

따라서 탐색·삽입·삭제 모두 $\Theta(\log n)$으로 보장된다. 대가는 각 노드가 높이(또는 균형 인수) 정보를 유지해야 하고, 삽입·삭제 때 회전이 필요하다는 점이다.

| 연산 | 평균 | 최악 |
|---|:---:|:---:|
| 탐색 | $\Theta(\log n)$ | $\Theta(\log n)$ |
| 삽입 | $\Theta(\log n)$ | $\Theta(\log n)$ |
| 삭제 | $\Theta(\log n)$ | $\Theta(\log n)$ |

---

## 5. 2-3 트리

### 정의

2-3 트리는 2-노드와 3-노드를 가질 수 있는 탐색 트리다. 2-노드는 일반 BST 노드처럼 키 하나와 자식 둘을 갖고, 3-노드는 키 두 개 $(k_1 < k_2)$와 자식 셋을 갖는다. 세 자식은 각각 $k_1$보다 작은 키, $k_1$과 $k_2$ 사이의 키, $k_2$보다 큰 키를 담는다.

2-3 트리는 높이 균형을 항상 유지한다. 즉 모든 잎이 같은 레벨에 있다. 이 불변식이 로그 높이를 보장하는 비결이다.

### 탐색과 삽입

탐색은 BST와 거의 같다. 각 노드에서 키들과 비교해 알맞은 자식으로 내려간다.

삽입은 언제나 잎에서 일어난다. 내려갈 잎을 찾은 뒤,

- 그 잎이 2-노드면 키를 추가해 3-노드로 만든다.
- 그 잎이 이미 3-노드면 키 세 개가 되어 넘치므로, 가운데 키를 부모로 올려 보내고(promote) 노드를 두 개로 분할한다(split). 부모도 넘치면 같은 분할이 위로 전파되며, 루트까지 전파되면 트리의 높이가 1 늘어난다.

가운데 키를 위로 올리는 분할 규칙 덕분에 모든 잎이 항상 같은 레벨에 머무른다. 트리가 위쪽으로만 자라기 때문이다.

### 삽입 예시

빈 2-3 트리에 $9, 5, 8$을 차례로 삽입하는 과정을 보자.

$9$를 넣으면 루트가 3-노드 $[9]$가 되고, $5$를 더하면 $[5, 9]$가 된다. 여기에 $8$을 넣으면 $[5, 8, 9]$로 넘치므로, 가운데 키 $8$을 위로 올리고 $5$와 $9$를 좌우 자식으로 분할한다.

$$[5, 8, 9] \;\Rightarrow\; \begin{array}{c} [8] \\ [5] \quad [9] \end{array}$$

![2-3 트리 분할](/assets/img/posts/twenty-three-tree-example.png)
_잎 $[5, 9]$에 $8$을 넣으면 키가 셋이 되어 넘친다(왼쪽). 가운데 키 $8$을 부모로 올리고 $5$와 $9$를 좌우로 분할하면 모든 잎이 같은 레벨에 머무는 트리가 된다(오른쪽)_

결과는 루트 $[8]$, 왼쪽 자식 $[5]$, 오른쪽 자식 $[9]$인 높이 균형 트리다.

### 효율 분석

원소가 $n$개인 2-3 트리의 높이 $h$는 다음 범위에 있다.

$$\log_3(n + 1) - 1 \le h \le \log_2(n + 1) - 1$$

즉 $h \in \Theta(\log n)$이며, 탐색·삽입·삭제 모두 최악의 경우에도 $\Theta(\log n)$을 보장한다. AVL 트리가 회전으로 균형을 유지한다면, 2-3 트리는 노드의 표현 자체(2-노드/3-노드)를 바꿔 같은 목표를 달성한다.

---

## 정리

| 트리 | 변형 | 균형 유지 방법 | 탐색·삽입·삭제 |
|---|---|---|:---:|
| BST | 표현 변경 | 없음 (퇴화 가능) | 평균 $\Theta(\log n)$ / 최악 $\Theta(n)$ |
| AVL 트리 | 인스턴스 단순화 | 회전으로 재균형 | 최악 $\Theta(\log n)$ |
| 2-3 트리 | 표현 변경 | 다중 키 노드 + 분할 | 최악 $\Theta(\log n)$ |

세 트리 모두 집합을 트리 표현으로 바꿔 사전 연산을 처리하지만, 최악의 경우를 다스리는 방식이 다르다. 다음 글에서는 표현 변경의 또 다른 사례인 힙과 힙정렬, 그리고 Horner의 법칙과 문제 축소를 다룬다.

---
