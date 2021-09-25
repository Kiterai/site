# common_range
* ranges[meta header]
* concept[meta id-type]
* std::ranges[meta namespace]
* cpp20[meta cpp]

```cpp
namespace std::ranges {
  template<class T>
  concept common_range = range<T> && same_as<iterator_t<T>, sentinel_t<T>>;
}
```
* range[link range.md]
* same_as[link /reference/concepts/same_as.md]
* iterator_t[link iterator_t.md]
* sentinel_t[link sentinel_t.md]

## 概要
`common_range`は、イテレータと番兵の型が等しいRangeを表すコンセプトである。

標準のコンテナはすべて`common_range`のモデルである。

## モデル
型`T`が`common_range`のモデルとなるのは、`T`が[`range`](range.md)のモデルであり、`T`から取得した番兵とイテレータの型が等しい場合である。

## 例
(執筆中)

### 出力
(執筆中)

## バージョン
### 言語
- C++20

### 処理系
- [Clang](/implementation.md#clang): 13.0.0
- [GCC](/implementation.md#gcc): 10.1.0
- [ICC](/implementation.md#icc): ??
- [Visual C++](/implementation.md#visual_cpp): 2019 Update 10

## 参照
- [N4861 24 Ranges library](https://timsong-cpp.github.io/cppwp/n4861/ranges)
- [C++20 ranges](https://techbookfest.org/product/5134506308665344)