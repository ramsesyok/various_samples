## よくある質問（FAQ）

- [**doctest と Catch の違いは？**](#doctest-と-catch-の違いは)
- [**doctest と Google Test の違いは？**](#doctest-と-google-test-の違いは)
- [**コンパイル時間のパフォーマンスを最大にするには？**](#コンパイル時間のパフォーマンスを最大にするには)
- [**doctest はスレッドセーフ？**](#doctest-はスレッドセーフ)
- [**モック（mocking）には対応している？**](#モックmockingには対応している)
- [**静的ライブラリに書いたテストが登録されないのはなぜ？**](#静的ライブラリに書いたテストが登録されないのはなぜ)
- [**C文字列（`char*`）の比較がポインタ比較になるのはなぜ？**](#c文字列charの比較がポインタ比較になるのはなぜ)
- [**ヘッダオンリーライブラリにテストを書くには？**](#ヘッダオンリーライブラリにテストを書くには)
- [**doctest は例外を使いますか？**](#doctest-は例外を使いますか)
- [**STL ヘッダでコンパイルエラーが出るのはなぜ？**](#stl-ヘッダでコンパイルエラーが出るのはなぜ)
- [**異なるバージョンの doctest を同じバイナリで使える？**](#異なるバージョンの-doctest-を同じバイナリで使える)
- [**なぜ doctest はマクロを使うの？**](#なぜ-doctest-はマクロを使うの)
- [**複数のソースファイルで使うには？**](#複数のソースファイルで使うには)

---

### doctest と Catch の違いは？

**doctest** の主な利点：

- [スレッドセーフ](#doctest-はスレッドセーフ)
- テスト外でもアサーションを使用可能
- ヘッダのインクルードによるコンパイル時間が [Catch の 1/20](https://github.com/doctest/doctest/blob/master/doc/markdown/benchmarks.md) 以下
- アサーション自体も Catch よりも高速にコンパイル可能
- 実行時間も Catch より高速な場合が多い
- `DOCTEST_CONFIG_DISABLE` を使えばテスト関連コードをすべてバイナリから除外可能
- ヘッダインクルードによって不要な依存が発生しない
- MSVC / GCC / Clang の厳しい警告レベルでも警告ゼロ
- 多数のコンパイラでテスト済み（180+）
- テストケースをヘッダに書いても重複登録されない
- 他の DLL や exe にテストランナーを移すことも可能

**Catch** にない機能としては：

- [テストスイート](testcases.md#test-suites)
- [デコレータ](testcases.md#decorators) など

一方、**doctest** に不足しているのは：

- マッチャー（Matchers）やジェネレータ
- マイクロベンチマーク（Catch は nonius を使用）
- タグ（擬似的にエミュレートは可能）

> doctest は Catch の「軽量で洗練されたサブセット」と言えます。

---

### doctest と Google Test の違いは？

主な違い：

- doctest はプロダクションコードのすぐ隣でも快適に使える（ヘッダオンリー・高速）
- doctest は単一ヘッダ、Google Test は静的ライブラリを別途ビルド・リンクが必要
- Subcase によって setup/teardown が柔軟に記述できる（Google Test は fixture が冗長）
- doctest の方がコンパイルも実行も速い
- Windows 含めてスレッドセーフ（Google Test は pthread 前提）
- 全体的に API がシンプル

doctest に足りない機能：

- 値パラメータ付きテスト
- デス（crash）テスト
- Google Mock との統合（doctest でも可能性はある）

---

### コンパイル時間のパフォーマンスを最大にするには？

- `DOCTEST_CONFIG_SUPER_FAST_ASSERTS` を定義するとアサーションのコンパイル時間が最大 91% 改善します。
- `CHECK_EQ` などの [バイナリアサーション](assertions.md#binary-and-unary-asserts) を使うとさらに高速化。

副作用：

- `try/catch` が除外されるため、例外が投げられると即座に test case が終了
- デバッガで停止すると doctest の内部で止まるため、一段 callstack を上がる必要あり

---

### doctest はスレッドセーフ？

はい。以下がスレッドセーフです：

- [アサーション](assertions.md)
- [ログ](logging.md)マクロ

ただし注意点もあります：

- test case の並列実行は非対応（直列実行）
- subcase はメインスレッドでのみ使うべき
- subcase 内でスレッドを立てる場合は、subcase 終了前に join すること
- コンテキストログはスレッドローカルで保持される

複数プロセスでの並列実行には [範囲実行スクリプト](../../examples/range_based_execution.py) を使います。

---

### モック（mocking）には対応している？

ネイティブ対応はしていませんが、以下のようなライブラリと連携可能です：

- [trompeloeil](https://github.com/rollbear/trompeloeil)
- [FakeIt](https://github.com/eranpeer/FakeIt)

失敗の記録には `ADD_FAIL_AT(file, line, message)` などの [ログマクロ](logging.md#messages-which-can-optionally-fail-test-cases) を使います。

---

### 静的ライブラリに書いたテストが登録されないのはなぜ？

**自己登録コードがあるライブラリ全般で起きる問題です。**

原因：静的ライブラリは使われるシンボルのみリンク対象になるため、テストがリンクされない可能性あり。

回避策：

- CMake の `OBJECT` ライブラリを使う：

  ```cmake
  add_library(with_tests OBJECT test1.cpp test2.cpp)
  add_executable(main $<TARGET_OBJECTS:with_tests> main.cpp)
  ```

- [`doctest_force_link_static_lib_in_target()`](../../examples/exe_with_static_libs/doctest_force_link_static_lib_in_target.cmake) を使う（ただしプリコンパイルヘッダ使用時など一部制限あり）

- MSVC の場合：`/OPT:NOREF` または `/WHOLEARCHIVE` を使う

---

### C文字列（`char*`）の比較がポインタ比較になるのはなぜ？

`char*` はポインタとして扱われるためです。  
`DOCTEST_CONFIG_TREAT_CHAR_STAR_AS_STRING` を定義すると `strcmp()` による比較になります。

---

### ヘッダオンリーライブラリにテストを書くには？

方法は2通り：

1. `doctest.h` をヘッダに含めてそのままテストを書く（使用側で test runner の実装が必要）
2. `#ifdef DOCTEST_LIBRARY_INCLUDED` でテスト部分を囲む（使用側が doctest を含んでいれば有効化される）

テストにはフィルター用タグをつけておくと便利（例：`TEST_CASE("[my_lib] ～")`）

---

### doctest は例外を使いますか？

はい。ただし例外なし環境でも動作可能です。  
`DOCTEST_CONFIG_NO_EXCEPTIONS` を定義してください。

---

### STL ヘッダでコンパイルエラーが出るのはなぜ？

標準準拠のためには `DOCTEST_CONFIG_USE_STD_HEADERS` を定義してください。  
標準ライブラリの forward 宣言との非互換が起きている可能性があります。

---

### 異なるバージョンの doctest を同じバイナリで使える？

現在は非対応です。  
内部的にはシングルトンやグローバルレジストリを使うためバージョンが混ざると不整合になります。

---

### なぜ doctest はマクロを使うの？

「マクロ＝悪」という考え方もありますが、テストフレームワークの DSL を実現するには適切です。  
Catch の作者 Phil Nash の解説が [こちら](http://accu.org/index.php/journals/2064) にあります。

---

### 複数のソースファイルで使うには？

1つの `.cpp` ファイルで以下のどちらかを定義してください：

- `DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN`（test runner も含む）
- `DOCTEST_CONFIG_IMPLEMENT`（test runner は自作）

他のソースファイルでは通常通り `#include "doctest.h"` を記述するだけでOKです。

---

[ホームへ戻る](readme.md#reference)
