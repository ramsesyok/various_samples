# チュートリアル
## Catch2 を入手する

最も簡単に Catch2 を入手する方法は、最新の [シングルヘッダ版](https://raw.githubusercontent.com/catchorg/Catch2/v2.13.10/single_include/catch2/catch.hpp) をダウンロードすることです。シングルヘッダは複数のヘッダを統合して生成されますが、通常のヘッダファイルのソースコードです。

他の方法としては、システムのパッケージマネージャーを使う、または [CMake パッケージ](cmake-integration.md#installing-catch2-from-git-repository) でインストールする方法もあります。

Catch2 の完全なソースコード（テストプロジェクト、ドキュメント、その他を含む）は GitHub にホストされています。[http://catch-lib.net](http://catch-lib.net) からリダイレクトされます。

## どこに置くか？

Catch2 はヘッダオンリーです。プロジェクトからアクセスできる場所にファイルを置くだけで使えます。ヘッダ検索パスを通すか、プロジェクトツリー内に直接配置しても構いません。特に、他のオープンソースプロジェクトで Catch を使ってテストスイートを構築する場合に便利です。詳しくは [このブログ記事](https://levelofindirection.com/blog/unit-testing-in-cpp-and-objective-c-just-got-ridiculously-easier-still.html) を参照してください。

このチュートリアルでは Catch2 のシングルインクルードヘッダ（または include フォルダ）が無修飾で使用できることを前提としていますが、必要であればフォルダ名のプレフィックスが必要になる場合があります。

_パッケージマネージャーや CMake パッケージで Catch2 をインストールした場合、`#include <catch2/catch.hpp>` の形式でインクルードしてください。_

## テストの記述

簡単な例から始めましょう（[コードはこちら](../examples/010-TestCase.cpp)）。階乗を計算する関数を書いたと仮定し、それをテストしたいとします（ここでは TDD は一旦置いておきます）。

```c++
unsigned int Factorial( unsigned int number ) {
    return number <= 1 ? number : Factorial(number-1)*number;
}
```

簡単にするため、すべてを1つのファイルにまとめましょう（<a href="#scaling-up">テストファイルの構成については後述</a>）。

```c++
#define CATCH_CONFIG_MAIN  // Catch に main() を提供させる - このマクロは1つの cpp ファイルのみに記述してください
#include "catch.hpp"

unsigned int Factorial( unsigned int number ) {
    return number <= 1 ? number : Factorial(number-1)*number;
}

TEST_CASE( "Factorials are computed", "[factorial]" ) {
    REQUIRE( Factorial(1) == 1 );
    REQUIRE( Factorial(2) == 2 );
    REQUIRE( Factorial(3) == 6 );
    REQUIRE( Factorial(10) == 3628800 );
}
```

このコードは、[コマンドライン引数](command-line.md)に対応した完全な実行ファイルとしてコンパイルされます。引数なしで実行すると、すべてのテストケース（この場合は1つ）を実行し、失敗の有無、合否の要約、失敗したテスト数を出力します（単純に「成功したかどうか」を知りたいときに便利です）。

このまま実行すればすべて通ります。完璧ですね。...本当に？

実は、まだバグがあります。このチュートリアルの初期版にも実際にバグがありました（ありがとう、Daryle Walker（```@CTMacUser```）！）。

何がバグかというと、ゼロの階乗はいくつでしょうか？
[ゼロの階乗は1](http://mathforum.org/library/drmath/view/57128.html) です ― これは常識として覚えておく必要があります。

テストケースを修正して追加しましょう：

```c++
TEST_CASE( "Factorials are computed", "[factorial]" ) {
    REQUIRE( Factorial(0) == 1 );
    REQUIRE( Factorial(1) == 1 );
    REQUIRE( Factorial(2) == 2 );
    REQUIRE( Factorial(3) == 6 );
    REQUIRE( Factorial(10) == 3628800 );
}
```

これで失敗するようになります。出力例：

```
Example.cpp:9: FAILED:
  REQUIRE( Factorial(0) == 1 )
with expansion:
  0 == 1
```

このように、```==``` 演算子を使った自然な表現でも、実際の返り値（この場合は 0）を表示してくれます。問題点が一目で分かります。

関数を以下のように修正しましょう：

```c++
unsigned int Factorial( unsigned int number ) {
  return number > 1 ? Factorial(number-1)*number : 1;
}
```

これで全てのテストが通るようになります。

もちろん、他にも問題はあります。例えば、戻り値が `unsigned int` の範囲を超えるとすぐに問題になります。階乗では簡単に起こります。そのようなケースもテストに追加して、どう扱うかを検討するとよいでしょう（ここでは省略します）。

### ここで何をしたのか？

このシンプルなテストを通して、Catch の使い方の基本がいくつか分かりました。先に進む前に少し振り返ってみましょう。

1. `#define` で識別子を1つ定義し、ヘッダを1つ `#include` するだけで、[コマンドライン引数に対応する](command-line.md) `main()` の実装も含め、すべてが得られました。この `#define` は1つの実装ファイルだけで使ってください（理由は明白でしょう）。複数のテストファイルがある場合は、単に `#include "catch.hpp"` するだけで OK です。通常は、`#define CATCH_CONFIG_MAIN` と `#include "catch.hpp"` だけの専用ファイルを1つ用意するのがよいでしょう。自分で `main()` を実装して Catch を手動で起動することも可能です（[独自の main() を提供する場合](own-main.md) を参照）。
2. テストケースは `TEST_CASE` マクロで記述します。このマクロは1つまたは2つの引数を取ります ― テスト名（自由形式）と、オプションで1つ以上のタグ（詳細は <a href="#test-cases-and-sections">テストケースとセクション</a> を参照）。テスト名は一意でなければなりません。タグやワイルドカード付きのテスト名で特定のテストを実行できます。詳細は [コマンドラインドキュメント](command-line.md) を参照してください。
3. 名前とタグは単なる文字列で、関数やメソッドを宣言したり、明示的に登録する必要はありません。内部的には、自動的に名前のついた関数が定義され、静的レジストリクラスによって登録されます。このように識別子名の制約を受けずにテスト名を定義できるのです。
4. アサーションは `REQUIRE` マクロで記述します。専用のマクロを使わず、C/C++ の自然な構文で条件を記述します。内部では式テンプレートを使って左右の値を取得し、レポートに出力します。他のアサーションマクロもありますが、この技術のおかげでその数は大幅に削減されています。

<a id="test-cases-and-sections"></a>
## テストケースとセクション

多くのテストフレームワークでは、クラスベースのフィクスチャ機構を提供しています。つまり、テストケースはクラスのメソッドとしてマッピングされ、共通のセットアップやクリーンアップ処理は `setup()` や `teardown()` メソッド（または C++ のようにデストラクタを持つ言語ではコンストラクタ/デストラクタ）で行います。

Catch はこの方法も完全にサポートしていますが、いくつかの問題があります。特に、コードを分割する必要があったり、セットアップの粒度が粗すぎることが問題になる場合があります。メソッドごとに異なるセットアップが必要だったり、複数段階のセットアップが必要なこともあるでしょう（このチュートリアルの後半でこの概念を詳しく説明します）。こうした問題により、NUnit を開発した James Newkirk はすべてを最初からやり直し、[xUnit を開発](http://jamesnewkirk.typepad.com/posts/2007/09/announcing-xuni.html)することになりました（[背景はこちら](http://jamesnewkirk.typepad.com/posts/2007/09/why-you-should-.html)）。

Catch は、NUnit や xUnit とは異なるアプローチを採っており、C++ および C 系の言語により自然にフィットする方法を提供しています。以下に例を示します（[コード](../examples/100-Fix-Section.cpp)）：

```c++
TEST_CASE( "vectors can be sized and resized", "[vector]" ) {

    std::vector<int> v( 5 );

    REQUIRE( v.size() == 5 );
    REQUIRE( v.capacity() >= 5 );

    SECTION( "resizing bigger changes size and capacity" ) {
        v.resize( 10 );

        REQUIRE( v.size() == 10 );
        REQUIRE( v.capacity() >= 10 );
    }
    SECTION( "resizing smaller changes size but not capacity" ) {
        v.resize( 0 );

        REQUIRE( v.size() == 0 );
        REQUIRE( v.capacity() >= 5 );
    }
    SECTION( "reserving bigger changes capacity but not size" ) {
        v.reserve( 10 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 10 );
    }
    SECTION( "reserving smaller does not change size or capacity" ) {
        v.reserve( 0 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 5 );
    }
}
```

それぞれの `SECTION` に対して `TEST_CASE` は冒頭から実行されます。したがって、各セクションに入る時点で、`size` が5で `capacity` が少なくとも5であることが保証されています。これらは冒頭の `REQUIRE` によって検証されているので、自信を持って使えます。

この仕組みは、`SECTION` マクロが if 文を含み、Catch にセクションを実行するか問い合わせることで実現されています。各 `TEST_CASE` の実行では、1つの末端セクション（他にネストされた `SECTION` を含まないセクション）のみが実行され、他はスキップされます。次の実行では別のセクションが実行され、すべてのセクションが一度ずつ実行されるまで繰り返されます。

このようにして、セットアップ/クリーンアップ方式に比べて改善された点が見えてきます。コードがインラインで書け、スタックが使えるのです。

さらに、セクションの強力な使い方として、連続した操作を検証したい場合があります。ベクタの例を続けると、現在の容量より小さい値で `reserve()` しても何も変わらないことを検証したいとき、以下のように自然に記述できます：

```c++
    SECTION( "reserving bigger changes capacity but not size" ) {
        v.reserve( 10 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 10 );

        SECTION( "reserving smaller again does not change capacity" ) {
            v.reserve( 7 );

            REQUIRE( v.capacity() >= 10 );
        }
    }
```

セクションは任意の深さでネスト可能です（スタックサイズの制限を除く）。各末端セクションは、他の末端セクションとは独立して1回だけ実行されます（他のセクションに影響を与えることはありません）。親セクションで失敗が発生した場合、ネストされたセクションは実行されません ― これは意図された動作です。


## BDDスタイル

テストケースやセクションに適切な名前を付けることで、BDD（Behavior-Driven Development）スタイルの仕様記述を実現できます。これは非常に便利な方法であったため、Catch においても第一級のサポートが追加されました。

シナリオは `SCENARIO`、`GIVEN`、`WHEN`、`THEN` マクロを使って記述できます。これらはそれぞれ `TEST_CASE` や `SECTION` に対応します。詳細は [テストケースとセクション](test-cases-and-sections.md) を参照してください。

ベクタの例は、以下のようにこれらのマクロを使って書き換えることができます（[コード例](../examples/120-Bdd-ScenarioGivenWhenThen.cpp)）：

```c++
SCENARIO( "vectors can be sized and resized", "[vector]" ) {

    GIVEN( "A vector with some items" ) {
        std::vector<int> v( 5 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 5 );

        WHEN( "the size is increased" ) {
            v.resize( 10 );

            THEN( "the size and capacity change" ) {
                REQUIRE( v.size() == 10 );
                REQUIRE( v.capacity() >= 10 );
            }
        }
        WHEN( "the size is reduced" ) {
            v.resize( 0 );

            THEN( "the size changes but not capacity" ) {
                REQUIRE( v.size() == 0 );
                REQUIRE( v.capacity() >= 5 );
            }
        }
        WHEN( "more capacity is reserved" ) {
            v.reserve( 10 );

            THEN( "the capacity changes but not the size" ) {
                REQUIRE( v.size() == 5 );
                REQUIRE( v.capacity() >= 10 );
            }
        }
        WHEN( "less capacity is reserved" ) {
            v.reserve( 0 );

            THEN( "neither size nor capacity are changed" ) {
                REQUIRE( v.size() == 5 );
                REQUIRE( v.capacity() >= 5 );
            }
        }
    }
}
```

このように記述したテストは、実行時に次のような形式でレポートされます：

```
Scenario: vectors can be sized and resized
     Given: A vector with some items
      When: more capacity is reserved
      Then: the capacity changes but not the size
```

---

<a id="scaling-up"></a>
## スケーリングアップ

このチュートリアルでは簡単にするため、すべてのコードを1つのファイルに収めました。これにより、Catch を素早く簡単に始められますが、現実的なテストが増えてくると、この方法は適切ではありません。

必要なのは、次のブロック（または [同等のコード](own-main.md)）を _1つだけ_ のソースファイルに含めることです：

```c++
#define CATCH_CONFIG_MAIN
#include "catch.hpp"
```

それ以外のテストファイルは何ファイルでも追加できます。プロジェクトの構成や好みに応じて自由に分割してください。追加ファイルには `#include "catch.hpp"` のみを記述し、`#define` は繰り返さないようにしてください。

実際には、`#define` を含むブロックは [専用のソースファイル](slow-compiles.md) に分けるのが通常おすすめです（コード例：[main](../examples/020-TestCase-1.cpp), [tests](../examples/020-TestCase-2.cpp)）。

**注意：** ヘッダファイル内にテストコードを書かないでください！

---

## 型パラメータ化されたテストケース

Catch2 のテストケースは、型によってもパラメータ化できます。これには `TEMPLATE_TEST_CASE` や `TEMPLATE_PRODUCT_TEST_CASE` マクロを使います。これらは `TEST_CASE` マクロと同じように動作しますが、指定されたすべての型や型の組み合わせに対して実行されます。

詳細は、[テストケースとセクション](test-cases-and-sections.md#type-parametrised-test-cases) にあるドキュメントを参照してください。

---

## 次のステップ

このチュートリアルでは、Catch をすぐに使い始められるよう、簡潔な紹介をしてきました。また、他のフレームワークとの主要な違いについても示しました。これでもう、すぐにテストを書き始める準備ができています。

もちろん、学ぶべきことはまだありますが、多くは必要に応じて習得していけるはずです。成長し続けている [リファレンスセクション](Readme.md) をぜひ参照してください。

---

[ホームへ戻る](Readme.md)
