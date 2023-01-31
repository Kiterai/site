# println
* print[meta header]
* std[meta namespace]
* function template[meta id-type]
* cpp23[meta cpp]

```cpp
namespace std {
  template <class... Args>
  void println(format_string<Args...> fmt,
               Args&&... args);             // (1) C++23

  template <class... Args>
  void println(FILE* stream,
               format_string<Args...> fmt,
               Args&&... args);             // (2) C++23
}
```
* format_string[link /reference/format/format_string.md.nolink]
* FILE[link /reference/cstdio/file.md.nolink]

## 概要
書式指定で出力する。この関数は、出力の末尾に改行コードが自動で付加される。

書式は[`std::format()`](/reference/format/format.md)関数のページを参照。

この関数は、[`std::printf()`](/reference/cstdio/printf.md.nolink)関数ライクな書式指定で引数を文字列化して出力する。

- (1) : 標準出力に、書式指定で出力する
- (2) : 指定された[`FILE`](/reference/cstdio/file.md.nolink)に、書式指定で出力する

この関数は、末尾に改行コードが付くことに注意。改行コードが不要な場合は、[`std::print()`](print.md)関数を使用すること。

[`std::ostream`](/reference/ostream/basic_ostream.md)から派生したクラスオブジェクトに対して出力したい場合は、[`<ostream>`](/reference/ostream.md)ヘッダの[`std::print()`](/reference/ostream/println.md)関数を使用すること。


## 効果
- (1) : 以下と等価：
    ```cpp
    println(stdout, fmt, std::forward<Args>(args)...);
    ```
    * stdout[link /reference/cstdio/stdout.md.nolink]
    * std::forward[link /reference/utility/forward.md]

- (2) : 以下と等価：
    ```cpp
    print(stream, "{}\n", format(fmt, std::forward<Args>(args)...));
    ```
    * print[link print.md]
    * format[link /reference/format/format.md]
    * std::forward[link /reference/utility/forward.md]


## 例
```cpp example
#include <print>

int main()
{
  std::println("Hello {} World", 42);
}
```
* std::println[color ff0000]

### 出力
```
Hello 42 World
```

## バージョン
### 言語
- C++23

### 処理系
- [Clang](/implementation.md#clang): ??
- [GCC](/implementation.md#gcc): ??
- [ICC](/implementation.md#icc): ??
- [Visual C++](/implementation.md#visual_cpp): ??


## 関連項目
- [`std::format()`](/reference/format/format.md)
- [`std::print()`](print.md)


## 参照
- [P2093R14 Formatted output](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2093r14.html)