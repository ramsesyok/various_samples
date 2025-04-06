# ChatGPTとのプロンプトの流れ
## Prompt
```
C++のテストフレームワークである Catch2 とdoctestの比較を行ってください。
機能面の違いを細かく比較してください。
一方で、ソフトウェア開発の初心者が扱う上での違いを指摘してください。
また、チームとして初心者を含めてテスト駆動開発の適用を考えています。
```

## ChatGPT

```
Catch2 と doctest の比較について、機能面の違いに加えて、初心者の扱いやすさや、チームでの TDD 適用時の観点での比較も含めて調査します。以下について確認させてください：

1. 現在使用している C++ のバージョンはありますか？（例: C++11, C++17, C++20など）
2. 使用しているビルドシステム（例: CMake, Makefile, Bazelなど）はありますか？
3. テスト対象のプロジェクトは主にコンソールアプリケーション、ライブラリ、GUIなど、どのような種類ですか？
4. テストの規模や複雑性（例: 単体テスト中心、統合テストあり、大規模プロジェクトなど）はどの程度ですか？

差し支えなければ教えてください。それによって、より実用的な比較が可能になります。
```

## Prompt

```
1. C++11以上での適用を考えています。
2. 使用するビルドシステムは、CMakeを考えています。（Makefileが一部ありますが・・。）
3. コンソールアプリケーションが主体になります。
4. テスト規模は単体テスト中心です。エビデンスを伴う単体テストを一部扱いますが、テスト駆動開発による適用の容易性を重視しています。
```

## ChatGPT

```
ありがとうございます。C++11 以上、CMake 使用、単体テスト中心で TDD を導入したいとのことで、Catch2 と doctest の機能比較、初心者への扱いやすさ、そしてチームでの TDD 適用の観点から詳細に調査します。

調査が完了しましたら、機能比較表と実用的な観点からのおすすめポイントも含めてご報告いたします。
```

# Catch2 と doctest の詳細比較（C++テストフレームワーク）

**概要:** Catch2 と doctest はどちらも C++11 以降で利用可能なヘッダオンリーの単体テストフレームワークです。CMake を用いたコンソールアプリケーション開発において広く使われており、それぞれ特徴があります。本比較では、(1) 機能面の違い、(2) ソフトウェア開発初心者への扱いやすさ、(3) チームで TDD（テスト駆動開発）を導入する際の重要ポイント、の観点から両者を詳細に比較します。

## 1. 機能面での違い

### テストケースの記述方法と BDD スタイル対応

Catch2 も doctest も基本的に**テストケースを関数のように記述**でき、名前とオプションでタグを指定するマクロを提供します。たとえば Catch2 では `TEST_CASE("テスト名", "[タグ]") { ... }` のように書き、doctest でも同様に `TEST_CASE("テスト名") { ... }` が使えます。どちらも**テスト関数は自動登録**されるため、メイン関数から個別に呼び出す必要がありません。

両フレームワークとも **BDD (Behavior-Driven Development) スタイル** をサポートしています。具体的には、「シナリオ(Given-When-Then)」形式のマクロが用意されており、Catch2 では `SCENARIO("シナリオ名")`, `GIVEN("前提")`, `WHEN("操作")`, `THEN("結果")` といったマクロ、doctest でも同様に `SCENARIO(...)`, `GIVEN(...)`, `WHEN(...)`, `THEN(...)` マクロが利用可能です ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=,THEN%28%20something)) ([Catch2/docs/test-cases-and-sections.md at devel · catchorg/Catch2 · GitHub](https://github.com/catchorg/Catch2/blob/devel/docs/test-cases-and-sections.md#special-tags#:~:text=,))。これらは内部的には通常のテストケースやセクション(後述)として扱われ、BDD形式の記述をより読みやすくするためのエイリアスに過ぎません。BDDマクロを用いることで、テスト内容をまるで仕様書のように表現でき、特にシナリオベースのテストを書く際に有用です。

**テストケース内の構造化**に関しても、Catch2 と doctest はほぼ同等の機能を提供します。Catch2ではテストケース内部で`SECTION("説明") { ... }`を使ってコードブロックを分岐させることができ、doctestでは同様の機能として`SUBCASE("説明") { ... }`が用意されています。このセクション/サブケース機能により、**1つのテストケース内で共通のセットアップをした上で複数のパスをテスト**できます。例えば「ファイルを開いて読めること」と「ファイルを開いて書き込めること」を同一のセットアップでテストする場合、1つのテストケース内で2つのSECTION/SUBCASEに分岐させることで、ファイルオープン処理を共有しつつ別々の検証を行えます。各セクションは独立して実行され、テスト実行モデルは深さ優先でトラバースされます（Catch2/doctest共通の仕様です）。

**コード例:** 以下に両フレームワークで同等のテストを書いたシンプルな例を示します。階乗関数`factorial()`の正しさを検証するテストケースです。

- *Catch2 の場合*（単一のヘッダ利用の場合は `#define CATCH_CONFIG_MAIN` を一度定義）:

  ```cpp
  #define CATCH_CONFIG_MAIN  // Catch2が自動でmain関数を生成
  #include <catch2/catch.hpp>

  int factorial(int n) {
      return n <= 1 ? 1 : factorial(n - 1) * n;
  }

  TEST_CASE("factorial 関数のテスト", "[factorial]") {
      REQUIRE(factorial(1) == 1);
      REQUIRE(factorial(2) == 2);
      REQUIRE(factorial(3) == 6);
      REQUIRE(factorial(10) == 3628800);
  }
  ```

- *doctest の場合*（単一ヘッダ`doctest.h`を使い、同様にmain関数を自動生成可能）:

  ```cpp
  #define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN  // doctestが自動でmain関数を生成
  #include "doctest.h"

  int factorial(int n) {
      return n <= 1 ? 1 : factorial(n - 1) * n;
  }

  TEST_CASE("factorial 関数のテスト") {
      CHECK(factorial(1) == 1);
      CHECK(factorial(2) == 2);
      CHECK(factorial(3) == 6);
      CHECK(factorial(10) == 3628800);
  }
  ```

ご覧のように、**記述方法は非常によく似ており**、TEST_CASEマクロやCHECK/REQUIREといったアサーションの使い方も共通しています。このデザインの共通性について、doctestの作者も「doctestはCatchの非常に洗練された軽量なサブセット（再実装）と考えられる」と述べています ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=But%20these%20things%20,are%20planned%20in%20the%20roadmap))。

### アサーションマクロとエラーメッセージの比較

どちらのフレームワークにも豊富な**アサーションマクロ**が用意されています。代表的なものは以下の通りです：

- **REQUIRE系**（Catch2）／**CHECK系**（doctestのCHECKはCatch2のREQUIRE相当）: 条件が満たされない場合にその場でテストを失敗させ中断するマクロ。`REQUIRE(a == b)` や `CHECK(x < y)` のように使用します。
- **CHECK系（Catch2）／WARN系（Catch2）**／**CHECK系（doctest）**: 条件が満たされない場合に失敗を記録しますが、テストの実行は継続します（non-fatal アサーション）。Catch2では `CHECK`、doctestでは `CHECK` がこれに相当します（doctestの`CHECK`は失敗時でも続行する点でCatch2のCHECKに対応します。doctestにも REQUIRE がありますが、実質的な動作はCHECK/REQUIREともCatch2と同様と考えてよいでしょう）。
- **例外の検証**: 両者とも `REQUIRE_THROWS(expr)` や `CHECK_THROWS_AS(expr, ExceptionType)` などのマクロで「特定の式が例外を送出する/しないこと」をアサートできます。
- **浮動小数点比較**: ULp精度やApproxによる近似比較機能。Catch2には `Approx` クラスやULP単位での比較機能があり、doctestでもCatch2の`Approx`と同等のものを`doctest::Approx`として利用できます ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=literal%20,name))（Catch2の内部実装を利用するため、doctestでは`using doctest::Approx;`とすれば同じApproxが使えます ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=literal%20,name))）。
- **マクロ名の衝突回避**: Catch2では内部実装上、`CATCH_REQUIRE`のような接頭辞付きマクロも提供されていますが、デフォルトで`REQUIRE`等もグローバルに定義されます。doctestでも同様に`DOCTEST_CHECK`等の形でマクロが定義されていますが、どちらも通常はシンプルなマクロ名で使用できます ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=Hamcrest%20matchers%20%28built,CHECK))。必要に応じて衝突を避ける設定も可能です。

エラーメッセージの**読みやすさ**に関して、Catch2と doctest は共通して**式の評価結果を自動的に分解して表示**する仕組みを持ちます。例えば `REQUIRE(x + y == 10)` が失敗した場合、どちらのフレームワークも「左辺の値 (x+y)」と「右辺の値 (10)」を詳細に表示してくれます ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=cases))。Catch2では「標準のC/C++演算子を用いて比較を書くが、失敗時には式全体が分解され各値が表示される」ことが特徴とされており ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,and%20rhs%20values%20are%20logged))、doctestもこれを踏襲しています。例えば出力例としては、Catch2/doctestのいずれでも:

```
test_case.cpp:10: FAILED:
  CHECK( x + y == 10 )
with expansion:
  CHECK( 3 + 5 == 10 )
```

のように、実際の値を展開したメッセージが表示されます。初心者にとってもどの値が間違っているのかが一目で分かるため、有用です。どちらも**カラー表示**による強調（失敗箇所を赤字など）もデフォルトで行われます。

一点、doctestのアサーションは**スレッドセーフ**であると明言されています ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,faster%20%20than%20%2066))。これは複数スレッドからアサーションマクロを呼んでも内部で競合状態が起きないことを意味します（もっとも、ユニットテストでは通常単一スレッドでテストを実行することが多いため、大きな差異ではありませんが、例えば並列にテストしたい特殊な場合には有利です）。

また、**アサーションの柔軟性**として doctest の特徴に「テストコンテキスト外でもアサーションを使える」点があります ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,faster%20%20than%20%2066))。doctestのマクロはテストケース内だけでなく通常の関数内でも利用可能で、テスト実行中でなければ単に何もしない（もしくは失敗時に例外を投げるのではなく、戻り値で結果を返す）動作をします ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=Pros%20of%20doctest%3A))。これにより、例えばサンプルコード中に doctest の `CHECK` を書いておいて、ドキュメントとして実行例検証に使う、といった使い方も可能です。Catch2にはこの機能はありません（テストケース外で呼ぶ用途は想定されていません）。

### タグ機能とテストのフィルタリング

**テストのタグ付けとフィルタリング**機能は、テストフレームワークを使う上で大規模テストの整理や一部実行に便利な機能です。Catch2はテストケースに**タグ**を付与する仕組みを持ち、TEST_CASEマクロの第二引数に文字列リテラルでタグを指定できます（例: `TEST_CASE("Foo機能のテスト", "[Foo][統合]")`）。タグは`[...]`で括った文字列として自由に設定でき、**コマンドライン引数**で実行するテストをフィルタリングする際に使用します。例えば `./tests [Foo]` のように実行すればタグ"Foo"を含むテストだけが、`~[統合]` とすればタグ"統合"が付いたテストを除外して実行する、といった指定が可能です ([c++ - Running unit tests excluding a specific tag in Catch2 - Stack Overflow](https://stackoverflow.com/questions/54693505/running-unit-tests-excluding-a-specific-tag-in-catch2#:~:text=,foo%5D%20tag))。Catch2はタグ表記による高度なフィルタ構文をサポートしており、複数タグのAND/OR指定やワイルドカード、否定(`~`)など柔軟に組み合わせて実行テストを指定できます ([c++ - Running unit tests excluding a specific tag in Catch2 - Stack Overflow](https://stackoverflow.com/questions/54693505/running-unit-tests-excluding-a-specific-tag-in-catch2#:~:text=,Other%20tests%20are%20not)) ([c++ - Running unit tests excluding a specific tag in Catch2 - Stack Overflow](https://stackoverflow.com/questions/54693505/running-unit-tests-excluding-a-specific-tag-in-catch2#:~:text=hidden%20test%20without%20,Selects%20tests%201%20and%204))。

doctestも**テストのグルーピング/フィルタ**機能を備えますが、そのアプローチは若干異なります。doctest本体には Catch2 のような明示的タグ文字列の概念は（少なくとも初期バージョンでは）ありませんでした。その代わりに**テストスイート (test suite)** 機能があり、`TEST_SUITE("suite名") { ... }` ブロックの中に書かれたテストケースは指定したスイート名に属するとみなされます ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=Test%20suites))。複数のテストケースを論理的にグループ化したい場合、suite名を付けておきコマンドライン引数で `--suite="suite名"` のように指定するとそのスイートに属するテストだけを実行できます。このほか doctestでは、テスト名やスイート名に対して**ワイルドカードや正規表現**によるフィルタもサポートしています（ドキュメントによれば、テスト選択フィルタで正規表現を使用可能） ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=Select%20tests%20by%20tag%20%E2%9C%93,tests%20%E2%9C%93%20%E2%9C%93%20%E2%9C%93%20%E2%9C%93))。一方、Catch2は正規表現はサポートせずワイルドカードマッチのみです。

Catch2のタグ機能に対応するものとして、doctestでも**タグをエミュレート**する方法が提供されています。doctestのFAQでは、Catchから移行する場合にテスト名文字列にタグを含め、それをフィルタに利用するテクニックが紹介されています ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=,see%20below)) ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=A%20quick%20and%20easy%20way,macros%20as%20follows))。実際、doctestの `TEST_CASE` マクロは内部的にテスト名にスイート名を連結できるようになっており、適切にマクロを再定義すれば Catch2 と同様の `[タグ]` 表記を扱うことも可能です ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=A%20quick%20and%20easy%20way,macros%20as%20follows))。

**テストのスキップ/条件実行**に関して、Catch2と doctest はやり方が異なります。Catch2ではタグ機能を利用し、「特殊タグ」として `[.]` を付けたテストケースはデフォルトでは実行されない（hidden扱い）というルールがあります ([Catch2/docs/test-cases-and-sections.md at devel · catchorg/Catch2 · GitHub](https://github.com/catchorg/Catch2/blob/devel/docs/test-cases-and-sections.md#special-tags#:~:text=%2A%20%60%5B.%5D%60%20,e.g.%20%60%5B.integration))。例えば `TEST_CASE("重いテスト", "[.][long]")` のように `[.]` を付与すると、通常実行では飛ばされます（実行したい場合は明示的にタグ指定する必要があります）。また Catch2 には `[!mayfail]` や `[!shouldfail]` といった特殊タグも定義されており、**意図的に失敗する可能性のあるテスト**や**失敗することを期待するテスト**をマークできます ([Catch2/docs/test-cases-and-sections.md at devel · catchorg/Catch2 · GitHub](https://github.com/catchorg/Catch2/blob/devel/docs/test-cases-and-sections.md#special-tags#:~:text=%2A%20%60%5B%21mayfail%5D%60%20,to%20track%20in%20your%20tests))。`[!mayfail]`が付いたテストは失敗してもテスト全体は失敗と見なされず、逆に`[!shouldfail]`が付いたテストは**成功すると**テスト全体としては失敗とみなされます ([Catch2/docs/test-cases-and-sections.md at devel · catchorg/Catch2 · GitHub](https://github.com/catchorg/Catch2/blob/devel/docs/test-cases-and-sections.md#special-tags#:~:text=%2A%20%60%5B%21mayfail%5D%60%20,to%20track%20in%20your%20tests))。これらは未完成のテストや一時的に無視したい失敗などに使われます。

doctestでも同様の機能を**デコレータ**という形で提供しています。doctestのテストケースは関数呼び出しのようにデコレータを付加でき、例えば `TEST_CASE("heavy test" * doctest::skip()) { ... }` と書くとそのテストケースをスキップに指定できます ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=supported%20decorators%3A))。さらに、`* doctest::may_fail()` や `* doctest::should_fail()` を付けてCatch2の`[!mayfail]`や`[!shouldfail]`相当の指定が可能です ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=%2A%20%60skip%28bool%20%3D%20true%29%60%20,progress%2C%20or%20a))。これらデコレータにより、タグベースではなくコード上で直感的にテストの性質を指定できるのが特徴です。また doctestには `* doctest::timeout(0.5)` のように**タイムアウト**（指定時間内に完了しなければ失敗）を付けるデコレータもあります ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=Test%20cases%20can%20be%20decorated,with%20additional%20attributes%20like%20this))。Catch2にはタイムアウト機能は組み込まれていないため、これはdoctest独自の便利機能と言えます。

**テスト実行のフィルタリング**はコマンドラインから行いますが、両者でオプションが多少異なります。Catch2は `--success` や `--duration` など多彩なオプションがありますが、doctestも `--success`（成功テストの表示）や`--no-skip`（skip指定を無視して実行）など類似のオプションを提供しています。基本的に、タグ/スイート以外の観点（ファイル名やテスト名の一部）でも実行テストを絞り込める点は共通しており、例えばCatch2ではファイル名をタグとして付与するオプション（実行時引数`-#`）もあります ([Catch2/docs/test-cases-and-sections.md at devel · catchorg/Catch2 · GitHub](https://github.com/catchorg/Catch2/blob/devel/docs/test-cases-and-sections.md#special-tags#:~:text=%2A%20%60%5B%23%3Cfilename%3E%5D%60%20,tags))。doctestでもテストケース名・スイート名・ソースファイル名でのフィルタ指定が可能です。

### フィクスチャ（セットアップ/ティアダウン）のサポート

**テストフィクスチャ**とは、複数のテストケース間で共有するセットアップ/後片付け処理を持つ仕組みです。Catch2も doctest も、フィクスチャを使ったテストを書くことができます。

1. **セクション/サブケースによる共有セットアップ:** これは前述のとおり、ひとつのテストケース内でSECTION(doctestではSUBCASE)を使う方法です。同じテスト関数の冒頭で共通の初期化処理を行い、SECTIONごとに異なる検証を記述することで、実質的に**共通セットアップ＋複数テスト**を実現します。Catch2の開発者も「セクションによってフィクスチャが不要になるケースが多い」と述べており ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=prefer%29)) ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,eliminates%20the%20need%20for%20fixtures))、doctestの作者も「サブケースはクラス継承によるフィクスチャよりもずっとクリーンな方法だ」 ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=static%20library%20and%20linked%20against,are%20available%20only%20on%20UNIX))としています。特に簡単な共有状態の扱いであれば、このセクション/サブケース機能で十分対応可能です。

2. **クラスを用いたフィクスチャ:** より伝統的な方法として、**テストクラスを継承してセットアップ/ティアダウン**を行う方法があります。Catch2では`TEST_CASE_METHOD(FixtureClass, "名前", "[タグ]")`マクロを使って、FixtureClassから派生したテストケースを作成できます（FixtureClassのコンストラクタでセットアップ、デストラクタで後処理を記述）。doctestでも同様に、`TEST_CASE_FIXTURE(FixtureClass, "名前")`マクロが用意されており、これを使うと内部的にFixtureClassを継承したクラスでテストが実行されます ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=TEST_CASE_FIXTURE%28UniqueTestsFixture%2C%20,))。FixtureClassのコンストラクタでDB接続を用意したり、ヘルパー関数をメンバとして用意しておけば、テストケース内でそれらを直接利用できます ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=The%20two%20test%20cases%20here,that%20any%20ID%27s%20created%20are))。以下は doctest の例です ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=TEST_CASE_FIXTURE%28UniqueTestsFixture%2C%20,)):

   ```cpp
   class UniqueTestsFixture {
   protected:
       DBConnection conn;
       static int uniqueID;
       UniqueTestsFixture() : conn(DBConnection::createConnection("myDB")) {}
       int getID() { return ++uniqueID; }
   };
   int UniqueTestsFixture::uniqueID = 0;

   TEST_CASE_FIXTURE(UniqueTestsFixture, "データベース挿入") {
       // この中では conn や getID() が使える
       REQUIRE_THROWS( conn.executeSQL("INSERT INTO employee (id, name) VALUES (?, ?)", getID(), "") );
   }
   ```

   Catch2でもほぼ同様の書き方が可能です。どちらのフレームワークも**タイプパラメータ付きのテスト**（テンプレートを用いたテスト）もサポートしています。Catch2では `TEMPLATE_TEST_CASE` マクロにより様々な型に対して同じテストを繰り返すことができますが、doctestも `TEST_CASE_TEMPLATE` や `SCENARIO_TEMPLATE` といったマクロを備えており、同様の機能を提供しています ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=))。例えば「int型とlong型で振る舞いが同じことを確認するテスト」を1度の記述で行う、といった用途に使えます。

   **パラメータ化テスト**（異なる入力値の組み合わせで同じテストを実行する）については、Catch2にはデータジェネレータ機能（`GENERATE`マクロ）があります ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,name%20clashes%20can%20be%20managed))。例えば `int x = GENERATE(1,2,3);` のように書くと、同じテストケースが x=1,2,3 でそれぞれ実行されます。doctestには組み込みのデータ生成機能はありません（FAQでも「ジェネレータは未実装」とされています ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=Missing%20stuff%3A))）。ですので、doctestで同様のことをしたい場合はforループで回すか、複数のTEST_CASEを書くかといった対応になります。

### テストディスカバリ（自動登録）と実行方法の違い

Catch2 と doctestはいずれも**ヘッダオンリー**のライブラリで、プロジェクトへの組み込みも似ています。どちらも「一つの翻訳単位に特別なマクロを定義してヘッダをインクルードする」ことで、自動的に`main()`関数を提供できます（Catch2では `CATCH_CONFIG_MAIN`、doctestでは `DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN` を定義） ([doctest/doc/markdown/tutorial.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/tutorial.md#:~:text=))。これにより、開発者は自前で`main()`を実装しなくてもテストを実行できる実行ファイルが得られます。カスタムの`main()`関数を使いたい場合、Catch2では `CATCH_CONFIG_RUNNER` を使ってCatch2の初期化だけリンクし、自分でRunnerを呼ぶことも可能ですし、doctestでも同様に自前`main()`からテストランナーを起動できます。

**テストケースの自動登録**は両者共通の特性で、リンクされたテストケースはプログラム起動時に自動で一覧に登録されます。各テストには名称がついており、実行時にはフレームワークが用意する**コマンドライン引数**でどのテストを実行するか指定可能です。例えば引数なしで起動すれば全テスト実行、`--list-test-cases`オプションで利用可能なテスト一覧を表示、`--success`で成功したテストも個別表示、といったオプションが多数用意されています ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,function))。基本的な**出力形式**はコンソール向けの人間に読みやすい書式ですが、Catch2は**レポーター (reporter)** と呼ばれる出力モジュールに基づいており、様々な出力形式を選択できます ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,the%20debugger%20on%20common%20platforms))。デフォルトは凝った複数行表示ですが、1行に結果をまとめるコンパクト表示、XML形式、JUnit XML形式、TAP形式、TeamCity用形式などを標準でサポートします ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,reporters%20can%20easily%20be%20added)) ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=Console%2C%20multi,%E2%9C%93%20%E2%9C%93))。対する doctest はシンプルなコンソール出力がメインで、現時点でXMLやJUnit形式の**組み込みレポーターはありません** ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=Console%2C%20multi,%E2%9C%93%20%E2%9C%93))。そのためCIツールとの連携では、Catch2は生成したJUnit XMLをCIに食わせるだけで済むのに対し、doctestでは標準出力をパースするスクリプトを書いたり、独自にレポータークラス（IReporter インターフェース）を実装してJUnit出力をさせる必要があります。もっとも、doctestでも **カスタムレポーター**を作成する仕組み自体は提供されています（IReporter interfaceを実装すればCatch2同様にイベントをフック可能）という情報もあります ([doctest - "The Lightest C++ Testing Framework" - for CI) : r/cpp - Reddit](https://www.reddit.com/r/cpp/comments/b4jo5u/doctest_the_lightest_c_testing_framework_version/#:~:text=doctest%20,))が、公式ドキュメント上はCatch2ほど豊富な組み込みレポーターは用意されていません。

**CIや開発ツールとの統合**の点では、Catch2がリッチな出力形式を持つこと以外にもいくつか違いがあります。たとえば、CMakeのCTestとの連携では、Catch2には `catch_discover_tests()` という便利な関数（CMakeモジュール）が用意されており、ビルド後にテスト実行ファイルを `--list-tests` モードで起動してCatch2のテストを自動登録する仕組みがあります ([Catch2/docs/cmake-integration.md at devel - GitHub](https://github.com/catchorg/Catch2/blob/devel/docs/cmake-integration.md#:~:text=Catch,test%20flag))。これはCTestで個々のCatch2テストをテストケース単位で扱えるようにする機能で、大規模プロジェクトで多数のテストを管理する際に便利です。doctestにも `--list-test-cases` 等で一覧を取得する機能自体はあるので、スクリプトで同様のことは可能ですが、Catch2ほど公式に統合された仕組みはありません。ただしJetBrainsのCLionのようなIDEでは、Catch2同様にdoctestのテストもネイティブにサポートしており、IDE上で個々のテストを実行・デバッグできるようになっています ([ 単体テストのチュートリアル  | CLion ドキュメント ](https://pleiades.io/help/clion/unit-testing-tutorial.html#:~:text=%E3%81%93%E3%81%AE%E3%83%81%E3%83%A5%E3%83%BC%E3%83%88%E3%83%AA%E3%82%A2%E3%83%AB%E3%81%A7%E3%81%AF%E3%80%81%E5%8D%98%E4%BD%93%E3%83%86%E3%82%B9%E3%83%88%E3%81%AE%E3%82%A2%E3%83%97%E3%83%AD%E3%83%BC%E3%83%81%E3%81%AE%E6%A6%82%E8%A6%81%E3%82%92%E7%A4%BA%E3%81%97%E3%80%81CLion%20%E3%81%A7%E3%82%B5%E3%83%9D%E3%83%BC%E3%83%88%E3%81%95%E3%82%8C%E3%81%A6%E3%81%84%E3%82%8B%204%20%E3%81%A4%E3%81%AE%E3%83%86%E3%82%B9%E3%83%88%E3%83%95%E3%83%AC%E3%83%BC%E3%83%A0%E3%83%AF%E3%83%BC%E3%82%AF%20,Test%E3%80%81Catch2%E3%80%81Doctest%29%20%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%E8%AA%AC%E6%98%8E%E3%81%97%E3%81%BE%E3%81%99%E3%80%82))。これはCLion内でCatch2/Doctestのランナーが内蔵されているためで、IDE利用者にとってはどちらを選んでも利便性に大差はありません。

**実行速度**については、後述するように両者のオーバーヘッドに差がありますが、テストそのものの実行機能としてはほぼ同等です。どちらもデフォルトでは単一スレッドでテストケースを順次実行します（テストケース間の並列実行などは標準機能としては持ちません）。途中で失敗した場合の挙動も似ており、致命的アサーション(REQUIRE相当)で失敗するとそのテストケースはそこで中断して次のテストケースに進みます。すべてのテストが通った場合はプロセスの終了コード0、1つでも失敗があれば終了コード1となる点も共通しているため、CIで実行する際の**終了ステータスによる合否判断**も同様に扱えます ([doctest/doc/markdown/tutorial.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/tutorial.md#:~:text=This%20will%20compile%20to%20a,did%20it%20work))。

### パフォーマンス（実行性能とコンパイル時間）

**実行時の性能**は、数十件程度のテストであればCatch2とdoctestの間で体感できる差はあまりありません。しかし、アサーションの実装やエラー発生時の制御方法に違いがあるため、**大規模なテストスイートでは差が現れる可能性**があります。doctestの作者は「doctestはCatchよりテスト実行が何倍も速い」と主張しています ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,faster%20%20than%20%2066))。具体的な数字として公開されているものは少ないですが、doctest FAQによれば「doctestはCatchよりもテストの実行が**数倍高速**」とのことです ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,faster%20%20than%20%2066))。この差が生まれる要因の一つとして、Catch2（特にv2系）は**例外を使った制御フロー**を多用していた点が挙げられます。Catch2はデフォルトでテストケース内のREQUIRE失敗時に内部で例外を投げて脱出する設計になっており、セクションのネストごとに例外のtryブロックが生成されます。一方、doctestはより低コストなやり方でこれを実現しているため、失敗が多発する状況などでオーバーヘッドが小さいと考えられます。ただし、実行時間についてはアサーション何百万回といった極端なケースでなければ大差はないとの意見もあります。実際、50,000個のアサートを含むようなテストをコンパイル・実行するベンチマークでは、doctestの実行時間はCatch2より**25%程度速かった**という報告もあります ([doctest – the Lightest C++ Unit Testing Framewor](https://accu.org/journals/overload/25/137/kirilov_2343/#:~:text=,faster%20than%20Catch))。したがって、**実行性能**だけを見るとdoctestに軍配が上がるものの、その差はユニットテストの規模によっては誤差範囲かもしれません。

むしろ大きな違いが現れるのは**コンパイル時間**の面です。Catch2 は便利な反面テンプレートメタプログラミングやマクロ展開が重厚で、ヘッダをインクルードするだけでビルド時間への影響が大きいことが知られています。doctestはこの点を強く意識して作られており、「Catch2に比べて**ヘッダのインクルードコストが20倍以上軽い**」とされています ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,by%20defining%20the%20DOCTEST_CONFIG_DISABLE%20identifier))。実際、ACCUで報告されたベンチマークでは、Catchを1ファイルにインクルードするだけで約250～460msのコンパイル時間を要したのに対し、doctestは約10ms程度で済んだとのことです ([doctest – the Lightest C++ Unit Testing Framewor](https://accu.org/journals/overload/25/137/kirilov_2343/#:~:text=,faster%20than%20Catch))。つまりヘッダ導入のオーバーヘッドは**25～50倍**の差があります。この差はテストファイルの数が増えるほど効いてきます。さらに、アサーションを多数含む場合でもdoctestはCatch2よりコンパイルが速く、例えば5万個のアサートを持つコードを全てコンパイルする場合、doctestは約60秒で完了したのに対しCatch2は約80秒かかったというデータがあります ([doctest – the Lightest C++ Unit Testing Framewor](https://accu.org/journals/overload/25/137/kirilov_2343/#:~:text=,faster%20than%20Catch))。**約25%**のビルド時間短縮ですが、テストコード量が多いほど恩恵は大きくなります。

> ※参考データ：doctestヘッダのインクルードは約10ms、Catch2では250–460ms ([doctest – the Lightest C++ Unit Testing Framewor](https://accu.org/journals/overload/25/137/kirilov_2343/#:~:text=,faster%20than%20Catch))。5万アサートのコンパイルはdoctest約60秒、Catch2約80秒 ([doctest – the Lightest C++ Unit Testing Framewor](https://accu.org/journals/overload/25/137/kirilov_2343/#:~:text=,faster%20than%20Catch))。

doctestがなぜこれほど軽量かというと、**内部実装を極力シンプルにし依存ヘッダも少なく**しているためです。doctestは自体がほとんど他のヘッダをincludeせず、必要な機能だけを内包しています ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20everything%20testing,framework%20will%20still%20register%20the))。Catch2もv3でだいぶ軽量化されたものの、それでもdoctestのコンパイル速度には及びません。特にプロジェクト規模が大きくテストコードも大量になると、Catch2では**「テストを書くとビルドが遅くなる」**という事態になりがちです（実際、ある開発者は「Catch2はコンパイルが遅すぎる」と感じ、doctestに乗り換えたとコメントしています ([doctest (the fastest feature-rich single-header testing framework ...](https://www.reddit.com/r/cpp/comments/hh917p/doctest_the_fastest_featurerich_singleheader/#:~:text=,wait%20to%20slow%20to%20compile))）。TDDでは頻繁にテスト→実装→ビルド→テスト…を繰り返すため、このコンパイル時間の差は開発速度に無視できない影響を与えます。

両者の**実行バイナリサイズ**についても触れると、doctestは非常に小さく、フレームワーク自身のコード行数(LOC)は約4,500行、Catch2 v2系は約8,800行と報告されています ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=Can%20provide%20canned%20main,18))。実際のテスト実行ファイルのサイズも、doctestでは不要ならテスト関連コードをすべて取り除ける（後述）こともあり、かなりコンパクトになります。

### その他の機能と拡張性

**モック(framework)のサポート:** Catch2も doctest もモック機能（関数呼び出しを偽物に差し替えるような機能）は組み込んでいません。C++ではGoogle Test用のGoogle Mockや、trompeloeilといった専用モックライブラリを組み合わせることが多いですが、Catch2/doctestどちらも外部モックライブラリとの併用は可能です。特にCatch2はtrompeloeilとの相性が良く、公式ドキュメントでも触れられていますが、doctestでも基本的には同様の手法でモックを使えます。

**マッチャ(matchers):** テストの期待値比較を表現するDSL的な「マッチャ」については、Catch2は標準でいくつかのHamcrest風マッチャ（文字列の部分一致やベクトルの要素検証等）をサポートしています ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,and%20full%20set%20of%20matchers))。またカスタムマッチャを自分で定義する仕組みも提供されています。doctestは執筆時点ではマッチャ機能を持っておらず、FAQでも「マッチャは未サポート（今後のロードマップ）」とされています ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=Missing%20stuff%3A))。したがって、doctestで複雑な条件を検証する場合は`CHECK()`内に複雑な論理を書くか、あるいは自前でヘルパー関数を用意してそのbool結果をCHECKする、といった形になります。

**プロパティベーステスト/ファジーテスト:** 両者とも組み込みサポートはありません。Catch2は有志による拡張や別ライブラリとの組み合わせで可能ですが、標準機能としてはデータジェネレータ止まりです。

**ベンチマーク:** Catch2はマイクロベンチマーク機能を持っており、`BENCHMARK`マクロで特定コードの実行時間を測定できます ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=%2A%20Hamcrest,complex%20properties))。これは元々"Nonius"というライブラリを統合したもので、簡易な性能測定に便利です。doctestにはベンチマーク機能が**ありません** ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=Missing%20stuff%3A))。ベンチマークも行いたい場合、doctestユーザは別途Nanobenchなどの専用ライブラリを使う必要があります。

**テストコードの無効化:** doctest特有の利点として、**テストコードをビルドから除外できる**機能があります。`DOCTEST_CONFIG_DISABLE` というマクロを定義してビルドすると、doctestのマクロがすべて無効化され、テストコードがコンパイルされなくなります ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=,and%20through%20valgrind%2Fsanitizers%2Fanalyzers))。これにより、本番リリース用ビルドではテスト部分を完全に取り除くことが容易です。Catch2には直接対応する機能はありませんが、テスト用の翻訳単位だけ別にしておき、リリース時にはそれをコンパイルしない、といった運用で対応します。doctestの場合は同じソースにテストを書いておいても定数定義一つで除去できるので、**ヘッダオンリーのライブラリにテストを埋め込んで配布**する、といった使い方も可能です ([doctest – the Lightest C++ Unit Testing Framewor](https://accu.org/journals/overload/25/137/kirilov_2343/#:~:text=,a%20library%20for%20developers%2C%20then)) ([doctest – the Lightest C++ Unit Testing Framewor](https://accu.org/journals/overload/25/137/kirilov_2343/#:~:text=conditionally%20use%20the%20tests%20like,are%20shipped%20to%20the%20customer))。Catch2でも技術的には可能ですが、重たいので非推奨とされています ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=Auto,to%20avoid%20long%20compilation%20times))。この違いは、**テストと実装コードの統合度**に関わるポイントです。

以上の機能比較を踏まえて、主要な違いを表にまとめます。

| 観点                     | **Catch2** (v2/v3)                                          | **doctest** (v2)                                        |
| ------------------------ | ---------------------------------------------------------- | ------------------------------------------------------- |
| **記述スタイル**         | TEST_CASE＋SECTION、BDDシナリオ(GIVEN/WHEN/THEN)サポート ([Catch2/docs/test-cases-and-sections.md at devel · catchorg/Catch2 · GitHub](https://github.com/catchorg/Catch2/blob/devel/docs/test-cases-and-sections.md#special-tags#:~:text=,))。<br>名前とタグを文字列指定。 | TEST_CASE＋SUBCASE、BDDシナリオ対応（Catch2と同等のマクロ） ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=,THEN%28%20something))。<br>テストスイート機能あり ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=Test%20suites))。 |
| **アサーション**         | CHECK/REQUIRE 系（豊富なマクロ）。<br>浮動小数Approx比較、カスタムマッチャ有 ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,and%20full%20set%20of%20matchers))。<br>失敗時は式と値を分解表示 ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=cases))。 | CHECK/REQUIRE 系（ほぼCatch2互換）。<br>Approx利用可（Catch2からimport） ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=literal%20,name))。<br>マッチャ機能なし（今後対応予定） ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=Missing%20stuff%3A))。 |
| **タグ機能**             | **タグ文字列**でグループ化・除外実行 ([c++ - Running unit tests excluding a specific tag in Catch2 - Stack Overflow](https://stackoverflow.com/questions/54693505/running-unit-tests-excluding-a-specific-tag-in-catch2#:~:text=,foo%5D%20tag))。特殊タグ[.]/[!mayfail]等あり ([Catch2/docs/test-cases-and-sections.md at devel · catchorg/Catch2 · GitHub](https://github.com/catchorg/Catch2/blob/devel/docs/test-cases-and-sections.md#special-tags#:~:text=%2A%20%60%5B.%5D%60%20,e.g.%20%60%5B.integration)) ([Catch2/docs/test-cases-and-sections.md at devel · catchorg/Catch2 · GitHub](https://github.com/catchorg/Catch2/blob/devel/docs/test-cases-and-sections.md#special-tags#:~:text=%2A%20%60%5B%21mayfail%5D%60%20,to%20track%20in%20your%20tests))。<br>コマンドラインで柔軟フィルタ（AND/OR, ~, *）。 | **テストスイート**単位でグループ化 ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=Test%20suites))。タグの概念は薄い（名称で代用可） ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=,see%20below))。<br>skipや期待失敗はデコレータ指定 ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=supported%20decorators%3A)) ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=%2A%20%60may_fail%28bool%20%3D%20true%29%60%20,notified%20of%20accidental%2C%20or%20third))。正規表現フィルタ対応 ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=Select%20tests%20by%20tag%20%E2%9C%93,tests%20%E2%9C%93%20%E2%9C%93%20%E2%9C%93%20%E2%9C%93))。 |
| **フィクスチャ**         | SECTIONで共通セットアップ可能、TEST_CASE_METHODでクラスフィクスチャ ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=TEST_CASE_FIXTURE%28UniqueTestsFixture%2C%20,))。<br>型パラメータテスト、データジェネレータ（GENERATE）あり ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,name%20clashes%20can%20be%20managed))。 | SUBCASEで共通セットアップ、TEST_CASE_FIXTUREでクラスフィクスチャ ([doctest/doc/markdown/testcases.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/testcases.md#:~:text=TEST_CASE_FIXTURE%28UniqueTestsFixture%2C%20,))。<br>型パラメータテストあり、データジェネレータなし ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=Missing%20stuff%3A))。 |
| **テスト発見と実行**     | 自動登録、`CATCH_CONFIG_MAIN`でmain生成可能。<br>豊富なコマンドラインオプション（リスト表示、フィルタ等） ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,function))。<br>多数のレポーター（JUnit XML他）標準サポート ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,reporters%20can%20easily%20be%20added))。 | 自動登録、`DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN`でmain生成 ([doctest/doc/markdown/tutorial.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/tutorial.md#:~:text=))。<br>コマンドラインオプションは基本的機能を網羅。<br>出力はコンソール中心（カラー対応）、XML等は自作必要 ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=Console%2C%20multi,%E2%9C%93%20%E2%9C%93))。 |
| **CI・ツール統合**       | JUnit出力でCI連携容易 ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,reporters%20can%20easily%20be%20added))。<br>CTest統合関数あり ([Catch2/docs/cmake-integration.md at devel - GitHub](https://github.com/catchorg/Catch2/blob/devel/docs/cmake-integration.md#:~:text=Catch,test%20flag))。<br>CLionなどIDEサポートあり。 | XML出力なし（独自レポーター実装は可能）。<br>CTest統合は手動で可能。<br>CLionがネイティブ対応。 |
| **性能 (実行/コンパイル)** | 実行オーバーヘッドはやや大きめ（例外使用等）。<br>ヘッダ重く大量テストでコンパイル時間増大。 | 実行オーバーヘッド極小 ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,faster%20%20than%20%2066))。<br>ヘッダ軽量で**ビルド高速**（Catch2の1/20～1/50） ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,by%20defining%20the%20DOCTEST_CONFIG_DISABLE%20identifier))。 |
| **その他**               | ベンチマーク機能あり ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=%2A%20Hamcrest,complex%20properties))。<br>テストを別翻訳単位にして本番ビルドから除外可能。 | ベンチマーク機能なし ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=Missing%20stuff%3A))。<br>マクロでテスト無効化可能（本番で完全除去） ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=,and%20through%20valgrind%2Fsanitizers%2Fanalyzers))。 |

## 2. 初心者にとっての扱いやすさ

**総じて、Catch2とdoctestの基本的な使い方に大きな難易度差はありません。** 両者とも「ヘッダをインクルードしてマクロでテストを書くだけ」で動作し、シンプルなテストを書くだけであれば数分のセットアップで始められます ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,your%20project%20and%20you%E2%80%99re%20away)) ([doctest/doc/markdown/tutorial.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/tutorial.md#:~:text=A%20simple%20example))。以下では、初心者の観点で考えられる幾つかのポイントについて比較します。

- **セットアップの容易さ:**  Catch2は単一ヘッダ版を提供しており、公式サイトから2つのファイルをダウンロードしてプロジェクトに追加するだけですぐ利用できます ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,your%20project%20and%20you%E2%80%99re%20away))。CMake利用時も、Catch2はCMakeのConfigファイル提供や`find_package(Catch2)`にも対応しているため、vcpkgやConan経由でインストールして使うことも容易です。doctestは**1つのヘッダファイルのみ**で構成されており（約**4500行** ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=Can%20provide%20canned%20main,18))）、これをプロジェクトに追加して`#include`するだけで使えます ([doctest/doc/markdown/tutorial.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/tutorial.md#:~:text=A%20simple%20example))。CMakeでも、例えばGitのsubmoduleとしてヘッダを取り込むか、あるいは単にソースツリーにコピーしてインクルードパスを通すだけで動きます。どちらもビルドに必要な依存ライブラリは**C++標準ライブラリのみ**で、追加のリンク設定も不要なので、初心者でもビルド設定でつまづきにくいと言えます。

- **テスト記述のわかりやすさ:**  初めて単体テストを書く初心者にとって重要なのは「テストコードの記述が直感的であるか」です。Catch2, doctestともに**自然なC++コードの延長**としてテストが書けます。例えば関数を呼び出して`==`で比較するといった記述がそのままASSERT条件になるため、特別なDSLを学ぶ必要がありません。実際、Catch2では**「アサーションマクロはひとつだけ（==など標準演算子で比較し、フレームワーク側が展開表示してくれる）」**ことを謳っており ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,and%20rhs%20values%20are%20logged))、doctestもCHECK/REQUIREで同様の書き心地です。BDDスタイルマクロも用意されていますが、必須ではなく通常のTEST_CASEだけでも十分記述可能です。従って、「とりあえず関数を呼び出して期待値と比較するテストを書きたい」という初心者のニーズには、どちらのフレームワークもシンプルに応えてくれます。

- **エラーメッセージの読みやすさ:**  初心者がテストを書いて失敗した場合、そのメッセージから不具合を特定できることが重要です。Catch2/doctestの失敗メッセージは前述のように非常に明解で、どの式が失敗しどの値とどの値が比較されたのかが一目瞭然です ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=cases))。例えば、「expected 10 but got 8」等と出ずに、「CHECK(x+y == 10) with expansion: CHECK(3+5 == 10)」のように**コードと実際の値**が対になって表示されます。これは初心者に限らず有用ですが、assertが失敗してプログラムが落ちただけではなく、**どのテストのどの部分が失敗したかを詳細に教えてくれる**点で、フレームワークを使う価値を感じられるでしょう。なお、どちらもデフォルトでは失敗したテストケースのみログに詳細が出ます。Catch2では全テスト成功時には何も表示しないか簡潔な成功メッセージのみ（オプションで詳細出力可）、doctestも似たような挙動です。

- **ドキュメントの充実度:** Catch2は公式のドキュメントサイト（Read the Docs）が充実しており、チュートリアルからリファレンスまで体系立てて学べます。また書籍やブログ、Qiita記事などコミュニティによる情報も豊富です。日本語情報もCatch(1.x)時代から一定数あり、検索すれば基本的な使い方程度なら見つかるでしょう。doctestはCatchほど知名度が高くないため日本語記事は少ないですが、公式リポジトリにMarkdown形式のドキュメント（FAQやチュートリアル）が用意されており、内容もCatch2を踏襲しているため英語が読めれば理解しやすいです。 ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=Aside%20from%20everything%20mentioned%20so,which%20%2066%20doesn%27t))に示されているとおり、Catchの設計にかなり沿っているので、Catch2のノウハウがそのままdoctestにも活かせます。またdoctest作者の執筆したACCU誌の論文やFAQ ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,by%20defining%20the%20DOCTEST_CONFIG_DISABLE%20identifier))で設計思想や使い方が語られており、情報源として参考になります。

- **サンプルコードの多さ:**  Catch2は多くのオープンソースプロジェクトで使われているため、GitHub上で検索すれば様々な実例を見ることができます。doctestは使用例が少なめですが、それでもGitHubで `#include "doctest.h"` を検索するといくつもプロジェクトが見つかります。doctestの公式にはシンプルなサンプルしかありませんが、Catch2のサンプルコードやCatchユーザの知見はほぼそのままdoctestにも応用できるため、「情報量」という観点でも致命的な差はないでしょう。

- **学習コスト:** 両者の基本が似ていることから、どちらか一方に慣れれば他方もすぐに習得できます。強いて言えばCatch2の方が機能が多いため、**全機能を把握しようとすると**学習範囲が広くなります。例えばタグや生成、マッチャ、ベンチマークなど、doctestには無い機能まで追い始めると結構な情報量です。しかし初心者が最初からそれら高度な機能を使うケースは稀で、まずは基本的なTEST_CASEとアサーションが使えれば十分でしょう。その意味では**どちらも入門は容易**で、差異はそれほど大きくありません。

- **トラブルシューティング:**  初学者にありがちな「コンパイルが通らない」「リンクエラーが出た」「テストを書いたのに実行されない」といった問題への情報提供も重要です。Catch2はFAQやStack OverflowでのQ&Aが豊富で、「TEST_CASEが実行されない場合は`CATCH_CONFIG_MAIN`を定義したファイルが含まれているか確認せよ」「マルチスレッドで初期化すると登録がうまくいかないことがある」等、情報が見つけやすい印象です。doctestもFAQで「なぜテストが見つからないのか（静的ライブラリ内に入れてしまうと自動登録されない）」などの注意点が説明されています ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=,executable%2Fdll))。利用者コミュニティの規模としてはCatch2の方が大きいため、日本語で質問した場合もCatch2の方が回答を得やすい可能性があります。

- **開発初心者への心理的ハードル:**  これは定性的な点ですが、Catch2は今やGoogleTestに次いで世界的に広く使われているフレームワークであり、「とりあえずこれを使っておけば間違いない」という安心感があります。JetBrainsのC++エコシステム調査(2021年)ではCatch2使用率が約11%で2番目に多いフレームワークだったとの報告もあります ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=Who%20else%20is%20using%20Catch2%3F))。一方、doctestは知る人ぞ知る存在で、「doctestを使っているよ」と言って通じる相手は限定されます。しかし機能が似通っているため、**初心者が最初に学ぶフレームワークとしてどちらが分かりやすいか**という観点では大差なく、「情報源の豊富さ」でCatch2がわずかに勝る程度でしょう。

まとめると、**初心者にとっては Catch2 も doctest も取り組みやすいテストフレームワーク**です。強いて言えば、doctestはシングルヘッダであることからビルドシステムの設定がさらに楽（ヘッダを1つ追加するだけ）という利点がありますが、Catch2もヘッダオンリーなのでほぼ同じです。また、doctestは実装コードと同じファイルにテストを書いて後から無効化できるため、小規模な学習プロジェクトでは「1ファイルに実装とテストを混在させてすぐ試せる」という手軽さがあります ([doctest – the Lightest C++ Unit Testing Framewor](https://accu.org/journals/overload/25/137/kirilov_2343/#:~:text=,a%20library%20for%20developers%2C%20then)) ([doctest – the Lightest C++ Unit Testing Framewor](https://accu.org/journals/overload/25/137/kirilov_2343/#:~:text=conditionally%20use%20the%20tests%20like,are%20shipped%20to%20the%20customer))。一方、Catch2はプロジェクトテンプレートや既存記事が豊富なので困ったときに調べやすい安心感があります。それぞれメリットはありますが、いずれにしても初心者が単体テストの基礎を学ぶ上で有用なツールと言えます。

## 3. チームで TDD を導入する際の重要ポイント

最後に、複数人の開発チームでTDDを実践するためにCatch2またはdoctestを導入する場合の考慮事項を比較します。チーム開発では単なる機能比較以上に、**開発フローへの馴染みやすさ**や**保守性・拡張性**、**学習コストの分散**といった側面が重要になります。

- **学習コストとチームの受け入れ:** 新たにテストフレームワークを導入する際、チームメンバーへの教育コストが問題になります。Catch2は前述の通り利用者が多く情報も豊富なため、メンバー各自が独学で習得しやすい環境があります。ドキュメントも整備されているので、新人メンバーに「公式チュートリアルを読んで」と指示すれば一通り学べるでしょう。一方、doctestはフレームワーク自体のシンプルさゆえに**学ぶべき事柄が少なく**、半日もあれば主要機能をマスターできる軽量さがあります。「Catch2相当のことがひととおりできる」ことを理解するのも容易です ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=But%20these%20things%20,are%20planned%20in%20the%20roadmap))。実際、Catchからdoctestへの移行ガイドでは、マクロ名を少し置換するだけで大部分がそのまま動くとされており ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=A%20quick%20and%20easy%20way,macros%20as%20follows))、**両者のインタフェースが類似**しているために学習コストの差は小さいです。したがって、チームにテストフレームワーク導入経験者がいない場合でも、doctestだから特別難しいということはなく、むしろ機能が絞られている分だけ覚えることが少ないメリットがあります。

- **TDDサイクルへの影響（ビルド＆テストのスピード）:** TDDではテストコードとプロダクションコードを頻繁に書き換えつつテストを実行するため、ビルドやテストの所要時間が開発スピードに直結します。前述したように、**doctest はコンパイル時間・実行時間ともに効率的**なので、大量のテストを書いてもビルドが極端に遅くなりにくいです ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,by%20defining%20the%20DOCTEST_CONFIG_DISABLE%20identifier)) ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=,faster%20%20than%20%2066))。一方、Catch2はテストケースやアサーションが増えるとビルド時間が長くなりがちで、「テストを書きすぎるとビルドが重い」というジレンマが生じる可能性があります。チーム全員がTDDを徹底する場合、例えばテストコードが数万行規模になってきても快適に回せるかは重要です。その点、doctestはテストを増やしてもビルド負荷が小さいため、**テストを書くこと自体がボトルネックになりにくい**という利点があります ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,by%20defining%20the%20DOCTEST_CONFIG_DISABLE%20identifier))。また、doctestなら不要になったテストを`DOCTEST_CONFIG_DISABLE`で一時的に無効化してビルド時間を短縮、といった柔軟な運用もできます ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=,and%20through%20valgrind%2Fsanitizers%2Fanalyzers))。もっとも、Catch2も通常の単体テスト規模（数百～数千件）であれば十分許容範囲のパフォーマンスではあります。チームのプロジェクト規模とテスト件数を見極めて、将来的に負荷が問題になりそうならdoctestを選ぶ、といった判断が考えられます。

- **チーム開発での運用のしやすさ:** 複数人でテストコードを書く際には、コードスタイル統一やテストの組織化がポイントです。Catch2はタグ機能が充実しており、大規模なテストでもタグでカテゴリ分けしておけば、テスト実行計画（例えば「ユニットテストだけ毎回実行、統合テストは夜間に回す」など）が立てやすいです。doctestでもスイート機能で似たことはできますが、タグに比べると自由度が低い面があります。もっとも、スイート名は文字列なのでチームでルールを決めればタグ同様に運用できます。またCatch2にはタグエイリアス機能 ([Catch2/docs/test-cases-and-sections.md at devel · catchorg/Catch2 · GitHub](https://github.com/catchorg/Catch2/blob/devel/docs/test-cases-and-sections.md#special-tags#:~:text=Between%20tag%20expressions%20and%20wildcarded,code%2C%20using%20the%20following%20form))もあり、複雑なフィルタ条件に名前を付けて再利用することもできます。これら高度な機能は大規模チームでこそ威力を発揮するものなので、テストが数十件程度ならどちらでも十分です。

- **CI/CD との統合:** 開発チームでは継続的インテグレーション(CI)環境でテストを自動実行し、結果をレポートすることが一般的です。Catch2は標準で**JUnit形式のレポート出力**が可能なため、CIサーバ上でテスト結果を集計・可視化するのが容易です ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,reporters%20can%20easily%20be%20added))。例えばJenkinsやGitLab CIのユニットテストレポート機能とすぐに連携できます。doctestで同様のことを行うには、自前でIReporterを実装してJUnit XMLを吐くようにするか、テスト実行後に出力ログをスクリプトでJUnit形式に変換する必要があります。チームにその実装負担をかけたくない場合、Catch2の方がすぐにCIパイプラインに組み込みやすいでしょう。ただしCI上で単にテストの成否だけ確認できれば良い（詳細なレポートは不要）という場合には、どちらでも問題ありません。どちらも**テスト失敗時に非ゼロ終了コードを返す**ため、CIジョブの成否検知には十分です。

- **IDEやツールのサポート:** チーム開発では開発者が使うエディタ/IDE環境も様々です。Catch2とdoctestはいずれもCLionでサポートされていますし ([ 単体テストのチュートリアル  | CLion ドキュメント ](https://pleiades.io/help/clion/unit-testing-tutorial.html#:~:text=%E3%81%93%E3%81%AE%E3%83%81%E3%83%A5%E3%83%BC%E3%83%88%E3%83%AA%E3%82%A2%E3%83%AB%E3%81%A7%E3%81%AF%E3%80%81%E5%8D%98%E4%BD%93%E3%83%86%E3%82%B9%E3%83%88%E3%81%AE%E3%82%A2%E3%83%97%E3%83%AD%E3%83%BC%E3%83%81%E3%81%AE%E6%A6%82%E8%A6%81%E3%82%92%E7%A4%BA%E3%81%97%E3%80%81CLion%20%E3%81%A7%E3%82%B5%E3%83%9D%E3%83%BC%E3%83%88%E3%81%95%E3%82%8C%E3%81%A6%E3%81%84%E3%82%8B%204%20%E3%81%A4%E3%81%AE%E3%83%86%E3%82%B9%E3%83%88%E3%83%95%E3%83%AC%E3%83%BC%E3%83%A0%E3%83%AF%E3%83%BC%E3%82%AF%20,Test%E3%80%81Catch2%E3%80%81Doctest%29%20%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%E8%AA%AC%E6%98%8E%E3%81%97%E3%81%BE%E3%81%99%E3%80%82))、Visual Studio Codeでも拡張機能やCMakeTools＋CTestで統合できます。Visual Studio（Windows）の場合、標準のテストエクスプローラーはGoogle Testや MSTest に対応していますが、Catch2/doctestは標準ではサポート外です。しかしResharper C++といった市販拡張を用いれば両者ともVS内で実行・結果確認が可能です ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=IDE%20integration%20%C2%A0%20%C2%A0%20%C2%A0,Creator%20%3F%20%3F%20%E2%9C%93)) ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=Visual,see%20issue))。従ってIDE統合という観点でも両者に大きな違いはありません（若干Catch2の方が認知度が高い分サポートするツールが多い程度です）。

- **フレームワークの拡張性と将来性:** チームで運用していく中で、「出力をもう少しカスタムしたい」「特定の処理を各テストケースの前後に挟みたい」といった要求が出ることがあります。Catch2は**リスナーやカスタムレポーター**の仕組みがあり、テストケースの開始・終了イベントにフックした処理を書いたり、出力様式を自由に拡張できます ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,the%20debugger%20on%20common%20platforms))。doctestも内部的にはIReporterが用意されているため拡張は可能ですが、提供されているAPIやドキュメントはCatch2ほど豊富ではありません。従って、フレームワーク自体を拡張して独自機能を追加するような場合にはCatch2の方が情報も多く無難です。もっとも通常の単体テスト用途であれば、そこまでの拡張は不要でしょう。

- **保守性・コミュニティ:** フレームワーク自体のメンテナンスという点では、Catch2は複数のコアメンバーやコミッターによって開発・保守されています。Issue対応や機能追加も継続的に行われています。一方、doctestは主に単独の作者によるプロジェクトで、コミット頻度はCatch2ほど高くないものの安定しています。双方オープンソース（Catch2はBSL、doctestはMITライセンス）なので将来にわたり利用可能ですが、**バス係数**を考えるとCatch2の方が人員が多く安心感はあります。もっともCatch2もPhil Nash氏という創始者の存在が大きく、組織バックアップがあるGoogleTestなどに比べればコミュニティ規模は中程度です。doctestも作者が大学講義など各所で紹介して普及に努めており、突然開発停止という懸念は低いでしょう。チームで採用する際には、万一将来片方の開発が止まっても**もう片方に移行しやすい**ことも心強い点です。前述の通りインタフェースが類似しているため、doctestからCatch2、またはCatch2からdoctestへ比較的スムーズに移行できます ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=A%20quick%20and%20easy%20way,macros%20as%20follows))。実際doctestのREADMEでもCatchとの比較表を示し、将来的に機能追加で差分が埋まる可能性にも触れています ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=But%20these%20things%20,are%20planned%20in%20the%20roadmap))。つまり、チームの要求が変わったとき（例えば「CIレポートが欲しくなった」→Catch2へ移行、あるいは「ビルド時間が厳しい」→doctestへ移行）にも対応可能です。

- **導入事例と信頼性:** チームメンバーやステークホルダーにとっては、「他にどんなプロジェクトで使われているか」も判断材料になるでしょう。Catch2はOSSでも商用プロジェクトでも多数の実績があります ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=A%20whole%20lot%20of%20people,most%20popular%20unit%20testing%20framework))。doctestはCatch2ほどの広まりはありませんが、組み込み系プロジェクトや競技プログラミングのコード検証などで愛用している開発者がいます。信頼性という点では、どちらも自分自身で多数のテストを書いて品質を確保していることが公表されています（Catch2もdoctestも自分自身を用いたセルフテストやCIを完備 ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=CI%20testing%20Travis%20%E2%9C%93%20%E2%9C%93,))）。実際、doctestは180以上の異なるビルド構成で毎回テストされ、ValgrindやSanitizerも通して品質を高めていると述べられています ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=where%20the%20library%20gets%20implemented,so%20tests%20end))。このように、**フレームワーク自体の品質保証**がしっかりしている点は、安心してチームに導入できる条件と言えます。

以上を踏まえると、**チームでTDDを導入する際のフレームワーク選択**は、以下のような指針が考えられます。

- *ビルド時間・開発サイクルの軽さを最重視する場合:* doctest が適しています。大規模プロジェクトでテスト数が膨大になってもビルドが速く、テスト実行のオーバーヘッドも小さいため、開発者体験が良好です ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,by%20defining%20the%20DOCTEST_CONFIG_DISABLE%20identifier))。テストコードをプロダクションコードと一緒に管理し、本番ビルドでは取り除くという運用も容易です ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=,and%20through%20valgrind%2Fsanitizers%2Fanalyzers))。組み込みシステムなど例外非対応環境でも動作可能な点 ([catch-lest-other-comparison/README.md at master · martinmoene/catch-lest-other-comparison · GitHub](https://github.com/martinmoene/catch-lest-other-comparison/blob/master/README.md#:~:text=C%2B%2B%20standard%20C%2B%2B98%20C%2B%2B98%20C%2B%2B11,))も強みです。

- *機能の充実や周辺ツール連携を重視する場合:* Catch2 が適しています。タグ付け・豊富なレポーター・ベンチマーク機能・マッチャなど、将来的に「この機能が欲しかった」が出てくる可能性があるなら、初めからCatch2を使っておけば安心です。CIへの統合もスムーズで、既存のナレッジやツール資産も活かしやすいです ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,reporters%20can%20easily%20be%20added)) ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=))。

- *チームのスキルセット:* もしメンバーに既にCatch2の経験者がいるならCatch2で統一する方が早いでしょうし、逆に「ヘッダ1個で完結する手軽さ」に魅力を感じるメンバーが多ければdoctestを試してみる価値があります。どちらにせよ、TDD導入初期はフレームワーク習熟よりもテストを書く文化を醸成することの方が重要です。その意味で、**使いやすいと思える方を選ぶのが一番**です。

- *将来の方向性:* Catch2とdoctestの差分は時間とともに変化する可能性があります。doctestはロードマップでCatch2にない機能（マッチャやデータジェネレータ等）の追加を計画しており ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=Missing%20stuff%3A))、今後さらに機能強化されるかもしれません。一方Catch2も性能改善や機能拡充が続いています。チームとして長期的にどのようなテストスタイルを目指すか（BDDを前面に出す、テストをドキュメントとして扱う、など）も選定の材料になるでしょう。

## まとめ

Catch2 と doctest はC++11以降の単体テストフレームワークとしていずれも優秀であり、多くの点で共通点があります。**Catch2は機能豊富でエコシステムとの親和性が高く、doctestはシンプルで高速軽量**という性格の違いがあります。 ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,by%20defining%20the%20DOCTEST_CONFIG_DISABLE%20identifier)) ([doctest – the Lightest C++ Unit Testing Framewor](https://accu.org/journals/overload/25/137/kirilov_2343/#:~:text=,faster%20than%20Catch)) 

機能面では、両者ともテストケース記述のしやすさやBDDスタイル対応、基本的なアサーション・フィクスチャ機能などは同等であり、初学者でも直感的に扱えます。一方でCatch2はタグ付けや多彩なレポート出力、ベンチマークまで備えるのに対し、doctestは必要最小限に留め効率を追求しています。特にコンパイル時間の差は顕著で、**大規模プロジェクトでは doctest の軽量さが武器**になります ([doctest/doc/markdown/faq.md at master · doctest/doctest · GitHub](https://github.com/doctest/doctest/blob/master/doc/markdown/faq.md#:~:text=%2A%20doctest%20is%20thread,by%20defining%20the%20DOCTEST_CONFIG_DISABLE%20identifier))。CI連携や高度なカスタマイズが必要な場合は Catch2 の安心感が勝るでしょう ([Why do we need yet another C++ test framework? - Catch2 documentation](https://catch2-temp.readthedocs.io/en/latest/why-catch.html#:~:text=,reporters%20can%20easily%20be%20added))。

ソフトウェア開発初心者にとっては、どちらのフレームワークも習得しやすく、テストを書く体験そのものは似ています。強いて言えば情報量の多いCatch2に軍配が上がりますが、doctestもCatch2ユーザの知見を流用できるため問題は少ないです。

チームでTDDを導入する場合は、プロジェクト規模・文化・重視点に応じて選択すると良いでしょう。ビルド高速化やテスト埋め込み重視ならdoctest、機能網羅性や将来の拡張性重視ならCatch2、といった基準が考えられます。いずれにせよ、どちらを選んでも**C++での単体テストを書く生産性は格段に向上**し、テスト駆動開発の下地を整えることができます。両者の違いを正しく理解した上で、自分たちのチームに最適な方を選定するとよいでしょう。

