# ほとんどの`volatile`を非推奨化
* cpp20[meta cpp]

## 概要

C++20より、`volatile`の本来の役割に照らして不正確、あるいは誤解を招く用法や無意味な用法について非推奨とされるようになる。

非推奨とされるだけで削除されてはいないが、おそらくコンパイラはそれらの用法について警告を発するようになる。そのような用法はバグの原因となり危険であるため可能な限り使用を避けるべきである。

## `volatile`

`volatile`な変数（メモリ領域）への1度のアクセスは正確に1度だけ行われる必要があり、0回にも2回にもなってはならない。そして、`volatile`領域へのアクセスはその順序がコード上の順序と一致する必要がある。

`volatile`の効果（保証）は単純にはこれだけである。

ただし、`volatile`はそのようなメモリアクセスが分割されない事は保証していない。`volatile`メモリ領域の個々のバイトに対しては正確に1度のアクセスが保証されるが、`volatile`領域全体を見たときにアクセスが1度だけになるとは限らない。

そして、非`volatile`領域と`volatile`領域へのアクセスの間の相対的な順序が前後しない事まで保証していない。すなわち、`volatile`変数へのアクセスと通常の変数へのアクセスは順番が入れ替わりうる。

また、`volatile`はC++メモリモデルの一部ではなく、`volatile`領域へのアクセス順序とはC++メモリモデルにおける観測可能な順序を意味しない。プロセッサはC++コード上での順序で読み取った`volatile`領域へのアクセス命令を、アウトオブオーダーで発行・実行することができる。メモリモデルにおいて動作が保証されている同期機構を用いない場合、あるコアにおける命令の実行順は、他のコア（あるいはプロセッサの外部）からは異なった順序で実行されたかのように観測されうる。

`volatile`は主として、プログラムの実行環境のハードウェアなどのプログラム外部の環境との通信手段の一つとして利用され、スレッド間のやりとりなどプログラム内部での通信の手段としては適さない。そのような`volatile`の正しい用法によるメモリの読み書きは、他のどの手段よりも移植性があり機能的にも優れており、言語機能として有用なものである。

## コア言語における非推奨化

### 複合代入演算子、インクリメント演算子

複合代入演算子とは`*= /= %=  += -= >>= <<= &= ^= |=`の10個の演算子のことで、ある操作とその結果の代入をまとめて行うような演算子のことである。

複合代入演算子およびインクリメント演算子は、「読み出し - 更新（処理） - 書き込み」という3つの操作を1文で行う。

```cpp
volatile int a = 0;
int b = 10;

a += b;
// これは以下と等価
// int tmp = a; 
// a = tmp + b;

++a;
// int tmp = a;
// a = tmp + 1;

a--;
// int tmp = a;
// a = tmp - 1;
```

複合代入演算子の左辺にある`volatile`変数、およびインクリメント演算子の`volatile`オペランドには2回のアクセス（読み込みと書き込み1回づつ）が発生するが、このアクセスは複合代入演算子やインクリメント演算子の見た目や一般的な理解とは必ずしも一致しない。

`volatile`変数においてはそのアクセス（読み書き）が重要であり、コード上での1回のアクセスは実行時にも1回だけアクセスされる必要がある。しかし、複合代入演算子およびインクリメント演算子のアクセス回数は多くのプログラマにとって曖昧であるか、誤解されている。

従って、算術型・ポインタ型の`volatile`変数に対する組み込みの複合代入演算子およびインクリメント演算子の使用はバグの元であるので、非推奨とされる。

この場合、これらの複合的な演算子を用いず、明示的に「読み出し - 更新 - 書き込み」を分けて書くことで`volatile`変数へのアクセスをコード上でも明確にする事が推奨される。

### 連鎖した代入演算子

代入演算子の一部の用法には、複合代入演算子・インクリメント演算子と同様の問題がある。

```cpp
volatile int a, b, c;

a = b = c = 10;
// このような順序でアクセスが発生する
// c = 10;
// b = c;
// a = b;
```

このような連なった代入演算子の用法においては、どの変数にどんな順番で何回アクセスされるのかが非常に分かりづらくなる。

`volatile`変数においてはそのアクセス（読み書き）が重要であり、コード上での1回のアクセスは実行時にも1回だけアクセスされ、かつその順番が前後してはならない。しかし、代入演算子を連鎖させた場合、そのアクセス回数および順序は非常に認識しづらくなる。

従って、非クラス型の`volatile`変数に対するこのような代入演算子の使用はバグの元であるので、非推奨とされる。

ただし、非推奨となるのは代入演算子の両端のオペランド以外に`volatile`変数が表れるケースである。

```cpp
volatile int v1, v2, v3;

v1 = v2 = v3 = 10; // NG（非推奨）

int n;
v1 = n = 10;      // OK

v1 = n = v3 = 10; // NG（非推奨）

v3 = 10;          // OK
v1 = v3;          // OK
v1 = n = v3;      // OK
```

### 関数引数と戻り値型

関数引数を`volatile`修飾することは、シグナルや`setjmp/longjmp`によって外部から変更されている可能性を示唆するために有効であり、引数の`const`修飾同様に関数定義内では明確な意味を持つ。

一方呼び出し側から見ると、参照・ポインタではない`volatile`引数の意味は不明瞭である。参照・ポインタではない関数引数が`volatile`修飾されている場合、その関数はシグナルハンドラや`setjmp/longjmp`と共に使用されるはずであり、呼び出し側にもそのような実装詳細の一部が漏洩してしまう。

また、呼出規約によっては一部の引数を配置するレジスタが`volatile`となる事があるが、呼出規約はC++コード上で意味を持たず、そのような呼び出し規約がマークされている関数は非`volatile`関数宣言と同様に扱われる。しかし、一部のレジスタが`volatile`である事はABIによって処理されている。

結局、関数引数の`volatile`修飾は有用ではないため非推奨とされる。関数の引数を`volatile`としたい場合、関数内で`volatile`ローカル変数に引数をコピーする事が推奨される。一部の実装では、そのようなコピーは省略され、オーバーヘッドとはならない。

```cpp
void f1(volatile int n);   // NG（非推奨）
void f2(volatile int* p);  // OK
void f3(volatile int& r);  // OK
void f4(int volatile * p); // OK
void f5(int volatile & r); // OK
void f6(int * volatile p); // NG（非推奨）
```

また、参照・ポインタではない関数戻り値型の`volatile`修飾は完全に意味を持たない。

例えば、ローカル`volatile`変数を返す場合、そのアクセスは関数リターン時に値をコピーするために一度実行されるが、コピーした後の値はもはや元の`volatile`領域とは別の場所にある。`volatile`において重要なのは特定領域へのアクセスであり、暗黙にそのようなコピーが走る事はほとんどの場合にプログラマの意図とは異なる。

ローカルの非`volatile`変数を`volatile`として返すことには意味がない。戻り値を`volatile`領域に配置したい場合、関数の呼び出し側で`volatile`変数に受ければよい。

そして、戻り値型の`volatile`修飾は容易に無視する事ができる。

このように、`volatile`戻り値型は無意味であるため、非推奨とされる。戻り値を`volatile`として扱いたい場合は、戻り値を`volatile`変数に受ければよい。

```cpp
volatile int  f1();   // NG（非推奨）
volatile int* f2();   // OK
volatile int& f3();   // OK
int volatile* f4();   // OK
int volatile& f5();   // OK
int* volatile f6();   // NG（非推奨）
```

ただし、関数引数・戻り値型いずれにしても、ポインタ・参照への`volatile`修飾は明確な意味を持ち有用である（値ではなく、領域に`volatile`とマークしているため）。従って、非推奨とされるのは関数引数・戻り値型へのトップレベル`volatile`修飾のみであって、`volatile`ポインタ・参照型は依然として許可される。


### 構造化束縛宣言

構造化束縛宣言にも`volatile`修飾を行う事ができるが、ここでのCV修飾は右辺にある式の結果である暗黙のオブジェクトに対して作用している。

右辺の式の結果が`std::tuple/std::pair`等の`tuple-like`な型のオブジェクトである場合、構造化束縛はまずその結果オブジェクトを`volatile`修飾して受けておき、その結果オブジェクトに対して`std::get`で要素の取得を行う。しかし、`std::get`には`volatile`オーバーロードが欠けており、コンパイルエラーを起こす。

一方、構造化束縛の残りのケース（配列・構造体）の場合は`std::get`を用いないためこのような問題は起こらない。

```cpp
auto f() -> std::tuple<int, int, double>;

volatile auto [a, b, c] = f();  // NG
// ここでは以下の様な事が行われている
// volatile auto tmp = f();
// std::tuple_element_t<0, decltype(tmp)> a = std::get<0>(tmp);

int array[3]{};

volatile auto [a, b, c] = array; // OK
// ここでは以下の様な事が行われている
// volatile int tmp[] = {array[0], array[1], array[2]};
// volatile int a = tmp[0];

static_assert(std::is_volatile_v<decltype(a)>); // OK
```

このような非一貫性の他にも、構造化束縛の裏で行われている事が`volatile`には適さない。

構造化束縛の`volatile`修飾はその右辺にある暗黙のオブジェクトに対して行われるが、その事は構文からは完全に隠蔽されている。右辺の式の結果オブジェクトも場合によってコピーされたり参照のまま利用されたりと、扱いが変化しうる。また、構造化束縛宣言に指定した変数名はコンパイラの扱いとしては変数名ではなく、右辺の暗黙のオブジェクト内の対応する要素にバインドされた名前でしかない。そのような名前に対する`volatile`の効果は不明瞭であり、右辺の式の直接の結果の型もその要素の型も`volatile`ではない場合には意味をなさない。

`volatile`においてはその領域へのアクセスが重要であり、1度のアクセスは正確に1度だけ行われる必要があり、その順序は前後してはならない。構造化束縛宣言はその裏側で多くの事が起こりそれは場合によって変化しうるが、そこでどのオブジェクトが`volatile`となりどのような順番でアクセスが発生するのかは非常に不透明である。

従って、構造化束縛宣言の`volatile`修飾は非推奨とされる。

構造化束縛した名前が`volatile`である必要がある場合は、分解対象の右辺の結果オブジェクトの各要素型をあらかじめ`volatile`修飾しておく事が推奨される。

```cpp
auto f() -> std::tuple<int, int, double>;
int array[3]{};

volatile auto [a, b, c] = f();   // NG（非推奨）
volatile auto [a, b, c] = array; // NG（非推奨）

auto g() -> std::tuple<volatile int*, volatile int*, volatile double&>;
auto [a, b, c] = g();  // OK
```

## ライブラリにおける非推奨化
（執筆中）

## 検討されたほかの選択肢

### 関数引数・戻り値型への`const`修飾

関数引数・戻り値型への`volatile`修飾が非推奨とされたのとほぼ同様の理由によって、`const`修飾も非推奨とする事が提案されていたが、合意が取れなかったため非推奨とはならなかった。

おそらく、間違っていたり意味がなかったとしても、`volatile`と比べて幅広く使用されているために非推奨とする事が忌避されたものと思われる。

## 備考

非推奨化で触れられてはいないが、`volatile`変数を並行処理の共有変数として使用することは常に間違っている。

## 関連項目
（執筆中）

## 参照

- [P1152R0 Deprecating `volatile`](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1152r0.html)
- [P1152R1 Deprecating `volatile`](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1152r1.html)
- [P1152R2 Deprecating `volatile`](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1152r2)
- [P1152R4 Deprecating `volatile`](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1152r4.html)
- [P1831R0 Deprecating `volatile`: library](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1831r0.html)
- [P1831R0 Deprecating `volatile`: library](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p1831r1.html)