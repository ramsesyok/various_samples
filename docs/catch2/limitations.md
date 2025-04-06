# 既知の制限事項

時間の経過とともに、Catch2 におけるいくつかの制限が明らかになってきました。これらの一部は実装上の事情で簡単には変更できないもの、一部は開発リソースの不足によるもの、また一部はサードパーティ製品のバグによるものです。

---

## 実装上の制限

### ループ内の `SECTION` のネスト

`SECTION` をループの中で使用する場合、**ループの各イテレーションごとに異なる名前**を与える必要があります。  
推奨される方法は、ループカウンターをセクション名に組み込むことです：

```cpp
TEST_CASE( "Looped section" ) {
    for (char i = '0'; i < '5'; ++i) {
        SECTION(std::string("Looped section ") + i) {
            SUCCEED( "Everything is OK" );
        }
    }
}
```

あるいは、まさにこの目的のために用意された `DYNAMIC_SECTION` マクロを使用する方法もあります：

```cpp
TEST_CASE( "Looped section" ) {
    for (char i = '0'; i < '5'; ++i) {
        DYNAMIC_SECTION( "Looped section " << i) {
            SUCCEED( "Everything is OK" );
        }
    }
}
```

---

### 最後のセクションが失敗すると再実行される可能性がある

テストケースの最後の `SECTION` が失敗した場合、そのテストが再実行される可能性があります。  
これは Catch2 が `SECTION` を実行時に動的に検出するためで、最後のセクションが `REQUIRE` などで中断された場合、Catch2 は他にセクションがないと認識できず、再度テストケースを実行します。

---

### MinGW / Cygwin におけるコンパイル（リンク）が非常に遅い

MinGW で Catch2 をコンパイルすると、特にリンク時に非常に時間がかかることがあります。  
これは MinGW の標準リンカの性能の問題によるものです。  
MinGW に `-fuse-ld=lld` オプションを指定して `lld` を使用するようにすれば、リンク時間は大幅に短縮されます。

---

## 機能上の制限

ここでは未実装の機能やその状況、回避策について説明します。

---

### スレッドセーフなアサーション { #thread-safe-assertions }

Catch2 のアサーションマクロはスレッドセーフではありません。  
つまり、Catch のテスト内でスレッドを使用することはできますが、**アサーションマクロ（`REQUIRE`, `CHECK` など）とやりとりできるのは 1 スレッドのみ**です。

以下のように、アサーションがメインスレッドのみで使われていれば OK です：

```cpp
std::vector<std::thread> threads;
std::atomic<int> cnt{ 0 };
for (int i = 0; i < 4; ++i) {
    threads.emplace_back([&]() {
        ++cnt; ++cnt; ++cnt; ++cnt;
    });
}
for (auto& t : threads) { t.join(); }
REQUIRE(cnt == 16);
```

しかし、各スレッド内で `CHECK` を使用するのは NG です：

```cpp
std::vector<std::thread> threads;
std::atomic<int> cnt{ 0 };
for (int i = 0; i < 4; ++i) {
    threads.emplace_back([&]() {
        ++cnt; ++cnt; ++cnt; ++cnt;
        CHECK(cnt == 16); // ←これはダメ
    });
}
for (auto& t : threads) { t.join(); }
REQUIRE(cnt == 16);
```

なお、C++11 にはスレッドセーフにするためのツールが揃っているため、将来的にはこの制限を取り除く予定です。

---

### テスト内でのプロセス分離

Catch はプロセスを分離してテストを実行する機能（例：fork）をサポートしていません。  
Windows では `fork` が使えず、完全なプロセス生成しかできないこと、またコードをクロスプラットフォームで維持したいという理由から、今後も実装にはかなりの時間が必要です。

---

### 複数テストの並列実行

Catch のテスト実行は完全に直列です。  
テストが遅くて並列化したい場合、次の 2 つの方法が取れます：

1. テストを複数の実行バイナリに分割し、それらを並列に実行する  
2. Catch の `--list-test-names-only` でテスト一覧を取得し、実行バイナリを複数回起動し、それぞれに異なるテストセットを実行させる

どちらも手間はありますが、並列実行は可能です。

---

## サードパーティのバグ

ここではコンパイラや標準ライブラリなど、Catch 以外の要因による既知のバグを紹介します。

---

### Visual Studio 2015 — `GENERATE` で文字列リテラルが推論されるとコンパイル失敗

以下のように文字列リテラルを `GENERATE` で使うと、VS2015 ではコンパイルエラーになります：

```cpp
TEST_CASE("Deducing string lit") {
    auto param = GENERATE("start", "stop"); // コンパイルエラー
}
```

回避策：`as<std::string>` などを使って型推論を明示します。

---

### Visual Studio 2017 — assert 内で raw 文字列リテラルを使うとコンパイルに失敗

VS2017 (VC15) には、生の文字列リテラルを `#` 演算子で文字列化するとエラーになるバグがあります：

```cpp
#define CATCH_CONFIG_MAIN
#include "catch.hpp"

TEST_CASE("test") {
    CHECK(std::string(R"("\)") == "\"\\");
}
```

回避策：`CATCH_CONFIG_DISABLE_STRINGIFICATION` を定義して、アサーションの文字列化を無効にする：

```cpp
#define CATCH_CONFIG_FAST_COMPILE
#define CATCH_CONFIG_DISABLE_STRINGIFICATION
#include "catch.hpp"

TEST_CASE("test") {
    CHECK(std::string(R"("\)") == "\"\\");
}
```

この設定を使うと、出力は以下のようになります：

```
PASSED:
  CHECK( Disabled by CATCH_CONFIG_DISABLE_STRINGIFICATION )
with expansion:
  ""\" == ""\"
```

---

### Visual Studio 2015 — アライメント関連のコンパイルエラー (C2718)

`declval<T>` を使うと、`T` のアライメント要件が満たせずにコンパイルエラーになるバグがあります。

回避策：対象型の `Catch::is_range` を明示的に特殊化します。

---

### Visual Studio 2015 — Debug モードで行番号がずれる

VS2015 には、Debug モードで `__LINE__` マクロの展開が誤るバグがあります。

回避策：Release モードでビルドする。

---

### Clang / G++ — 例外の後にセクションがスキップされる

一部の `libc++` や `libstdc++` のバージョンには、例外が処理された後でも `std::uncaught_exception()` が `true` を返し続けるバグがあります。これにより、一部の `SECTION` がスキップされます。

例：

```cpp
TEST_CASE("a") {
    CHECK_THROWS(throw 3);
}

TEST_CASE("b") {
    int i = 0;
    SECTION("a") { i = 1; }
    SECTION("b") { i = 2; }
    CHECK(i > 0);
}
```

Clang + libc++ 環境でのみセクションがスキップされるなどの場合、このバグの可能性があります。  
回避策：修正版の標準ライブラリを使う。

---

### Clang / G++ — `Matches` マッチャーが常に false を返す

`libstdc++-4.8` にバグがあり、`<regex>` にあるすべてのマッチング関数が常に false を返します。  
`Matches` マッチャーは内部で `<regex>` を使っているため、これが影響します。

回避策：新しいバージョンの `libstdc++` を使用してください。

---

### libstdc++ と `_GLIBCXX_DEBUG` マクロ使用時にランダム順序でクラッシュ

Catch2 を `_GLIBCXX_DEBUG` を有効にした状態で `--order rand` を付けて実行すると、自己代入によるチェックが発動しクラッシュすることがあります。  
[これは libstdc++ 側の既知のバグです](https://stackoverflow.com/questions/22915325/avoiding-self-assignment-in-stdshuffle/23691322)

回避策：`--order rand` を使用しない。

---

[ホームに戻る](Readme.md)