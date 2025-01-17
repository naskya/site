# コンストラクタ
* queue[meta header]
* std[meta namespace]
* priority_queue[meta class]
* function[meta id-type]

```cpp
explicit priority_queue(
             const Compare& x = Compare(),
             const Container& other = Container());       // (1) C++03

explicit priority_queue(const Compare& x = Compare(),
                        Container&& y = Container());     // (1) C++11

priority_queue() : priority_queue(Compare()) {}           // (1) C++20

explicit priority_queue(const Compare& x)
  : priority_queue(x, Container()) {}                     // (2) C++20

priority_queue(const Compare& x, const Container& other); // (3) C++11

priority_queue(const priority_queue&);                    // (4) C++03
priority_queue(const priority_queue&) = default;          // (4) C++11

template <class InputIterator>
priority_queue(InputIterator first, InputIterator last,
               const Compare& x = Compare(),
               const Container& other = Container());     // (5) C++03

template <class InputIterator>
priority_queue(InputIterator first, InputIterator last,
               const Compare& x,
               const Container& other);                   // (6) C++11

template <class InputIterator>
priority_queue(InputIterator first, InputIterator last,
               const Compare& x = Compare(),
               Container&& other = Container());          // (7) C++11

priority_queue(priority_queue&&) = default;               // (8) C++11

template <class Alloc>
explicit priority_queue(const Alloc& alloc);              // (9) C++11

template <class Alloc>
priority_queue(const Compare& x, const Alloc& alloc);     // (10) C++11

template <class Alloc>
priority_queue(const Compare& x,
               const Container& other,
               const Alloc& alloc);                       // (11) C++11

template <class Alloc>
priority_queue(const Compare x&,
               Container&& other,
               const Alloc& alloc);                       // (12) C++11

template <class Alloc>
priority_queue(const priority_queue& que,
               const Alloc& alloc);                       // (13) C++11

template <class Alloc>
priority_queue(priority_queue&& que,
               const Alloc& alloc);                       // (14) C++11
```

## 概要
- (1) : デフォルトコンストラクタ
    - C++03 : 比較関数と元となるコンテナをコピーして構築する。
    - C++11 : 比較関数をコピー、元となるコンテナをムーブして構築する。
    - C++20 : (2)に移譲。
- (2) : 比較関数のコピーと元となるコンテナをデフォルト構築して構築するコンストラクタ。
- (3) : 比較関数と、元となるコンテナのコピーから構築するコンストラクタ。
- (4) : コピーコンストラクタ
- (5), (6), (7) : イテレータ範囲で優先順位付きキューを構築する。
- (8) : ムーブコンストラクタ
- (9) : アロケータを受け取るコンストラクタ
- (10) : 比較関数とアロケータを受け取るコンストラクタ
- (11) : 比較関数、元となるコンテナのコピー、アロケータを受け取るコンストラクタ
- (12) : 比較関数、元となるコンテナの一時オブジェクト、アロケータを受け取るコンストラクタ
- (13) : アロケータ指定でコピー構築する
- (14) : アロケータ指定でムーブ構築する


## 要件
`Compare`型パラメータ`x`が、[狭義の弱順序](/reference/algorithm.md#strict-weak-ordering)で定義されていること。


## 効果
- (1) :
    - C++03
        1. メンバ変数`comp`を`x`でコピー構築する。
        2. メンバ変数`c`を`other`でコピー構築する。
        3. [`make_heap`](/reference/algorithm/make_heap.md)`(c.begin(), c.end(), comp)`を呼び出す。
    - C++11
        1. メンバ変数`comp`を`x`でコピー構築する。
        2. メンバ変数`c`を`y`でムーブ構築する。
        3. [`make_heap`](/reference/algorithm/make_heap.md)`(c.begin(), c.end(), comp)`を呼び出す。
- (3) :
    1. メンバ変数`comp`を`x`でコピー構築する。
    2. メンバ変数`c`を`other`でコピー構築する。
    3. [`make_heap`](/reference/algorithm/make_heap.md)`(c.begin(), c.end(), comp)`を呼び出す。
- (4) :
    1. メンバ変数`comp`を`x`でコピー構築する。
    2. メンバ変数`c`を`other`でムーブ構築する。
    3. [`make_heap`](/reference/algorithm/make_heap.md)`(c.begin(), c.end(), comp)`を呼び出す。
- (5):
    1. メンバ変数`comp`を`x`でコピー構築する。
    2. メンバ変数`c`を`other`でコピー構築する。
    3. `c.insert(c.end(), first, last)`を呼び出す。
    4. [`make_heap`](/reference/algorithm/make_heap.md)`(c.begin(), c.end(), comp)`を呼び出す。
- (6) :
    1. `c.insert(c.end(), first, last)`を呼び出す。
    2. [`make_heap`](/reference/algorithm/make_heap.md)`(c.begin(), c.end(), comp)`を呼び出す。
- (7):
    1. メンバ変数`comp`を`x`でコピー構築する。
    2. メンバ変数`c`を`other`でムーブ構築する。
    3. `c.insert(c.end(), first, last)`を呼び出す。
    4. [`make_heap`](/reference/algorithm/make_heap.md)`(c.begin(), c.end(), comp)`を呼び出す。
- (9) :
    1. メンバ変数`c`のメモリアロケートに`alloc`を使用する。
    2. メンバ変数`comp`を値初期化する。
- (10) :
    1. メンバ変数`c`のメモリアロケートに`alloc`を使用する。
    2. メンバ変数`comp`を`x`で初期化する。
- (11) :
    1. メンバ変数`comp`を`x`でコピー構築する。
    2. メンバ変数`c`を`other`でコピー構築する。
    3. メンバ変数`c`のメモリアロケートに`alloc`を使用する。
    4. [`make_heap`](/reference/algorithm/make_heap.md)`(c.begin(), c.end(), comp)`を呼び出す。
- (12) :
    1. メンバ変数`comp`を`x`でコピー構築する。
    2. メンバ変数`c`を`other`でムーブ構築する。
    3. メンバ変数`c`のメモリアロケートに`alloc`を使用する。
    4. [`make_heap`](/reference/algorithm/make_heap.md)`(c.begin(), c.end(), comp)`を呼び出す。
- (13) :
    1. メンバ変数`comp`を`que.comp`でコピー構築する。
    2. メンバ変数`c`を`que.c`でコピー構築する。
    3. メンバ変数`c`のメモリアロケートに`alloc`を使用する。
    4. [`make_heap`](/reference/algorithm/make_heap.md)`(c.begin(), c.end(), comp)`を呼び出す。
- (14) :
    1. メンバ変数`comp`を`que.comp`でムーブ構築する。
    2. メンバ変数`c`を`que.c`でムーブ構築する。
    3. メンバ変数`c`のメモリアロケートに`alloc`を使用する。
    4. [`make_heap`](/reference/algorithm/make_heap.md)`(c.begin(), c.end(), comp)`を呼び出す。


## 例
```cpp example
#include <iostream>
#include <queue>
#include <vector>
#include <string>

template <class PriorityQueue>
void pop_print(const std::string& name, PriorityQueue& que)
{
  std::cout << name << " : ";
  while (!que.empty()) {
    std::cout << que.top() << ' ';
    que.pop();
  }
  std::cout << std::endl;
}

int main()
{
  // デフォルト構築
  std::priority_queue<int> que1;

  // que1からコピー構築
  std::priority_queue<int> que2 = que1;

  // que2からムーブ構築
  std::priority_queue<int> que3 = std::move(que2);

  // イテレータ範囲から構築
  const std::vector<int> v = {3, 1, 4};
  std::priority_queue<int> que4(v.begin(), v.end());

  // イテレータ範囲、比較関数オブジェクト、コンテナから構築
  const std::vector<int> v2 = {5, 2};
  std::priority_queue<int> que5(v.begin(), v.end(), {}, v2);

  pop_print("que3", que3);
  pop_print("que4", que4);
  pop_print("que5", que5);
}
```
* que.empty()[link empty.md]
* que.top()[link top.md]
* que.pop()[link pop.md]
* std::move[link /reference/utility/move.md]

### 出力
```
que3 : 
que4 : 4 3 1 
que5 : 5 4 3 2 1 
```

### 処理系
- [Clang](/implementation.md#clang): ??
- [GCC](/implementation.md#gcc): 4.7.0(アロケータ付き初期化以外は使用可能)
- [ICC](/implementation.md#icc): ??
- [Visual C++](/implementation.md#visual_cpp): ??


## 参照

- [P0935R0 Eradicating unnecessarily explicit default constructors from the standard library](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0935r0.html)
