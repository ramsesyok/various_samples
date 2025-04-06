# なぜ私のテストはコンパイルに時間がかかるのか？

Catch を使って書いたテストコードが、予想よりもずっとコンパイルに時間がかかると報告する人がいます。なぜでしょうか？

Catch は完全にヘッダーだけで実装されています。これにより若干のオーバーヘッドは発生しますが、思っているほど大きくはありません。そして、以下のようにテストコードを整理することで、それを最小限に抑えることができます。

---

## <a id="short-answer"></a>短い答え

1 つだけのソースファイルで、```#define CATCH_CONFIG_MAIN``` もしくは ```CATCH_CONFIG_RUNNER``` を ```#include``` よりも前に定義してください。そしてそのファイルには **テストケースを書かないこと！**  
多くの場合、このファイルは 2 行（`#define` と `#include`）だけになります。

---

## <a id="long-answer"></a>詳しい説明

通常、C++ のコードは宣言やプロトタイプを含むヘッダーファイルと、定義・実装を含む .cpp ファイルに分かれています。  
各 .cpp ファイルは、それがインクルードするすべてのヘッダーを展開した「翻訳単位」となり、コンパイラに渡されてオブジェクトファイルへとコンパイルされます。

しかし、関数やメソッドはヘッダーファイルに `inline` として書くこともできます。  
問題は、そうした定義がインクルードされるすべての翻訳単位でコンパイルされてしまうことです。

Catch は **完全にヘッダーで実装されている** ため、「すべての翻訳単位で Catch 全体をコンパイルしているのでは？」と思うかもしれません。  
実際にはそれほど悪くありません。Catch は内部的に `#ifdef` によって実装コードを分離し、それが **定義された 1 つの翻訳単位** にのみコンパイルされるようになっています。

その「定義された 1 つの翻訳単位」というのが、```CATCH_CONFIG_MAIN``` もしくは ```CATCH_CONFIG_RUNNER``` を定義したソースファイル（＝メインソースファイル）です。

結果として、そのメインソースファイルだけが Catch の本体をコンパイルします。  
したがって、**このファイルには Catch の定義とインクルード以外は何も書かないのがベスト** です。  
テストケースは別ファイルに書きましょう。  
そうすれば Catch 全体の再コンパイルを避けられ、ビルド時間が大きく短縮されます。

---

## <a id="practical-example"></a>実用的な例

チュートリアルの [`Factorial`](tutorial.md) 関数が `factorial.cpp` にあり、その宣言が `factorial.h` にあるとします。  
これをテストする際、そしてテストを増やしてもビルド時間が長くならないようにするには、以下の 2 つのファイルに分けます：

```cpp
// tests-main.cpp
#define CATCH_CONFIG_MAIN
#include "catch.hpp"
```

```cpp
// tests-factorial.cpp
#include "catch.hpp"
#include "factorial.h"

TEST_CASE( "Factorials are computed", "[factorial]" ) {
    REQUIRE( Factorial(1) == 1 );
    REQUIRE( Factorial(2) == 2 );
    REQUIRE( Factorial(3) == 6 );
    REQUIRE( Factorial(10) == 3628800 );
}
```

まず `tests-main.cpp` を 1 回だけコンパイルしておけば、以降は `tests-factorial.cpp` だけを再コンパイルしてリンクすれば済みます。  
テストを増やしても `tests-main.cpp` を再コンパイルしないで済むため、ビルド時間が大幅に削減されます。

```bash
$ g++ tests-main.cpp -c
$ g++ factorial.cpp -c
$ g++ tests-main.o factorial.o tests-factorial.cpp -o tests && ./tests -r compact
Passed 1 test case with 4 assertions.
```

その後、たとえば `REQUIRE( Factorial(0) == 1)` を追加した場合は `tests-factorial.cpp` のみを再コンパイル：

```bash
$ g++ tests-main.o factorial.o tests-factorial.cpp -o tests && ./tests -r compact
tests-factorial.cpp:11: failed: Factorial(0) == 1 for: 0 == 1
Failed 1 test case, failed 1 assertion.
```

---

## <a id="using-the-static-library-catch2withmain"></a>Catch2WithMain 静的ライブラリの使用

Catch2 にはランナー実装済みの静的ライブラリ `Catch2WithMain` も提供されています。  
ただし、Catch2 v2 の実装と C++ のリンク制約との関係で、**この機能は実験的** です。

`Catch2::Catch2WithMain` はサブディレクトリとして使っても、インストールされたライブラリとして使っても構いません。

### CMake 使用例
```cmake
add_executable(tests-factorial tests-factorial.cpp)
target_link_libraries(tests-factorial Catch2::Catch2WithMain)
```

### Bazel 使用例
```python
cc_test(
    name = "hello_world_test",
    srcs = [
        "test/hello_world_test.cpp",
    ],
    deps = [
        "lib_hello_world",
        "@catch2//:catch2_with_main",
    ],
)
```

---

## <a id="other-possible-solutions"></a>その他の可能な解決策

Catch のコンパイル時間を短縮するために、いくつかの機能を犠牲にするという選択肢もあります。  
詳細は [Catch のコンパイル時設定に関するドキュメント](configuration.md#other-toggles) を参照してください。

---

[ホームに戻る](Readme.md)