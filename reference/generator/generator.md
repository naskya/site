# generator
* generator[meta header]
* class template[meta id-type]
* std[meta namespace]
* cpp23[meta cpp]

```cpp
namespace std {
  template<class Ref, class V = void, class Allocator = void>
  class generator : public ranges::view_interface<generator<Ref, V, Allocator>> {
    ...
  }
}
```
* ranges::view_interface[link /reference/ranges/view_interface.md]


## 概要
`generator`クラステンプレートは、[コルーチン](/lang/cpp20/coroutines.md)の評価により生成される要素列のビュー(view)を表現する。
特殊化された`generator`は[`view`](/reference/ranges/view.md)および[`input_range`](/reference/ranges/input_range.md)のモデルである。

戻り値型`generator`のコルーチン（以下、ジェネレータコルーチン）では`co_yield`式を用いて値を生成する。`co_yield` [`std::ranges::elements_of`](/reference/ranges/elements_of.md)`(rng)`式を用いると、ジェネレータコルーチンから入れ子Range(`rng`)の各要素を逐次生成する。
ジェネレータコルーチンでは`co_await`式を利用できない。

ジェネレータコルーチンは遅延評価される。ジェネレータコルーチンが返す`generator`オブジェクトの利用側（以下、呼び出し側）で先頭要素[`begin`](generator/begin.md)を指す[イテレータ](generator/iterator.md)を間接参照するまで、ジェネレータコルーチンの本体処理は実行されない。
呼び出し側がイテレータの間接参照を試みるとジェネレータコルーチンを再開(resume)し、ジェネレータコルーチン本体処理において`co_yield`式に到達すると生成値を保持して再び中断(suspend)する。呼び出し側ではイテレータの間接参照の結果として生成値を取得する。


### 説明用メンバ
`generator`では下記の説明用メンバ型を定義する。

- `value` : [`conditional_t`](/reference/type_traits/conditional.md)`<`[`is_void_v`](/reference/type_traits/is_void.md)`<V>,` [`remove_cvref_t`](/reference/type_traits/remove_cvref.md)`<Ref>, V>`
- `reference` : [`conditional_t`](/reference/type_traits/conditional.md)`<`[`is_void_v`](/reference/type_traits/is_void.md)`<V>, Ref&&, Ref>`
- [`iterator`](generator/iterator.md) : ジェネレータが返すイテレータ型。

`generator`および[`promise_type`](generator/promise_type.md)の動作説明のため、下記の説明用メンバを用いる。

- [`coroutine_handle`](/reference/coroutine/coroutine_handle.md)`<`[`promise_type`](generator/promise_type.md)`>` : コルーチンハンドル(`coroutine_`)
- [`unique_ptr`](/reference/memory/unique_ptr.md)`<`[`stack`](/reference/stack/stack.md)`<`[`coroutine_handle<>`](/reference/coroutine/coroutine_handle.md)`>>`: アクティブスタック(`active_`)


## 適格要件
- テンプレートパラメータ`Allocator`が`void`ではない場合、[`allocator_traits`](/reference/memory/allocator_traits.md)`<Allocator>::pointer`はポインタ型であること。
- 説明用のメンバ型`value`はCV修飾されないオブジェクト型であること。
- 説明用のメンバ型`reference`は参照型、または[`copy_constructible`](/reference/concepts/copy_constructible.md)のモデルであるCV修飾されないオブジェクト型であること。
- 説明用のメンバ型`reference`が参照型の場合には`RRef` == [`remove_reference_t`](/reference/type_traits/remove_reference.md)`<reference>&&`、それ以外の場合には`RRef` == `reference`としたとき、それぞれ下記のモデルであること。
    - [`common_reference_with`](/reference/concepts/common_reference_with.md)`<reference&&, value&>`
    - [`common_reference_with`](/reference/concepts/common_reference_with.md)`<reference&&, RRef&&>`
    - [`common_reference_with`](/reference/concepts/common_reference_with.md)`<RRef&&, const value&>`

テンプレートパラメータ`Allocator`が`void`ではない場合、`Cpp17Allocator`の要件を満たすこと。


## メンバ関数
### 構築・破棄

| 名前            | 説明           | 対応バージョン |
|-----------------|----------------|----------------|
| [`(constructor)`](generator/op_constructor.md) | コンストラクタ | C++23 |
| `(destructor)` | デストラクタ | C++23 |
| `operator=(generator other) noexcept;` | ムーブ代入演算子 | C++23 |

### イテレータ
| 名前            | 説明           | 対応バージョン |
|-----------------|----------------|----------------|
| [`begin`](generator/begin.md) | Viewの先頭を指すイテレータを取得する | C++23 |
| [`end`](generator/end.md) | Viewの番兵を取得する | C++23 |

## メンバ型

| 名前            | 説明        | 対応バージョン |
|-----------------|-------------|-------|
| `yielded`      | `co_yield`式の引数型（後述） | C++23 |
| [`promise_type`](generator/promise_type.md) | ジェネレータコルーチンのPromise型 | C++23 |

``` cpp
using yielded =
  conditional_t<is_reference_v<reference>, reference, const reference&>;
```
* conditional_t[link /reference/type_traits/conditional.md]
* is_reference_v[link /reference/type_traits/is_reference.md]


## 例
### 例1: 単一値の生成
```cpp example
#include <generator>
#include <ranges>
#include <iostream>

// 偶数値列を無限生成するコルーチン
std::generator<int> evens()
{
  int n = 0;
  while (true) {
    co_yield n;
    n += 2;
  }
}

int main()
{
  // ジェネレータにより生成されるレンジのうち
  // 先頭から5個までの要素値を列挙する
  for (int n : evens() | std::views::take(5)) {
    std::cout << n << std::endl;
  }
}
```
* std::generator[color ff0000]
* co_yield[link /lang/cpp20/coroutines.md]
* std::views::take[link /reference/ranges/take_view.md]

#### 出力
```
0
2
4
6
8
```

### 例2: レンジ要素値の逐次生成
```cpp example
#include <generator>
#include <iostream>
#include <list>
#include <ranges>
#include <vector>

// レンジの要素値を逐次生成するコルーチン
std::generator<int> ints()
{
  int arr[] = {1, 2, 3};
  co_yield std::ranges::elements_of(arr);
  std::vector<int> vec = {4, 5, 6};
  co_yield std::ranges::elements_of(vec);
  std::list<int> lst = {7, 8, 9};
  co_yield std::ranges::elements_of(lst);
}

int main()
{
  for (int n : ints())) {
    std::cout << n << ' ';
  }
}
```
* std::generator[color ff0000]
* co_yield[link /lang/cpp20/coroutines.md]
* std::ranges::elements_of[link /reference/ranges/elements_of.md]

#### 出力
```
1 2 3 4 5 6 7 8 9 
```

### 例3: ジェネレータのネスト
```cpp example
#include <generator>
#include <iostream>
#include <ranges>
#include <memory>

// 二分木ノード
struct node {
  int value;
  std::unique_ptr<node> left = nullptr;
  std::unique_ptr<node> right = nullptr;
};

// 二分木を幅優先走査: 左(left)→自ノード→右(right)
std::generator<int> traverse(const node& e)
{
  if (e.left) {
    co_yield std::ranges::elements_of(traverse(*e.left));
  }
  co_yield e.value;
  if (e.right) {
    co_yield std::ranges::elements_of(traverse(*e.right));
  }
}

int main()
{
  // tree:
  //    2
  //   / ¥
  //  1   4
  //     / ¥
  //    3   5
  node tree = {
    2,
    std::make_unique<node>(1),
    std::make_unique<node>(
      4,
      std::make_unique<node>(3),
      std::make_unique<node>(5)
    ),
  };

  for (int n: traverse(tree)) {
    std::cout << n << std::endl;
  }
}
```
* std::generator[color ff0000]
* co_yield[link /lang/cpp20/coroutines.md]
* std::ranges::elements_of[link /reference/ranges/elements_of.md]
* std::make_unique[link /reference/memory/make_unique.md]

#### 出力
```
1
2
3
4
5
```


## バージョン
### 言語
- C++23

### 処理系
- [Clang](/implementation.md#clang):
- [GCC](/implementation.md#gcc):
- [ICC](/implementation.md#icc):
- [Visual C++](/implementation.md#visual_cpp):


## 関連項目
- [`std::ranges::elements_of`](/reference/ranges/elements_of.md)


## 参照
- [P2502R2 `std::generator`: Synchronous Coroutine Generator for Ranges](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2502r2.pdf)
