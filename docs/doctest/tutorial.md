# チュートリアル（Tutorial）

**doctest** を始めるのに必要なのは、[**最新版のヘッダファイル**](https://raw.githubusercontent.com/doctest/doctest/master/doctest/doctest.h)をダウンロードしてソースファイルにインクルードするだけです（またはこのリポジトリを Git サブモジュールとして追加しても構いません）。

このチュートリアルでは、以下のようにヘッダを直接使える状態であることを前提とします：  
```cpp
#include "doctest.h"
```  
つまり、`doctest.h` がテストコードと同じディレクトリにあるか、ビルドシステムでインクルードパスを正しく設定してある必要があります。

> ※ このチュートリアルでは [テスト駆動開発（TDD）](https://ja.wikipedia.org/wiki/テスト駆動開発) については扱いません。

---

## シンプルな例

たとえば、次のような `factorial()` 関数をテストしたいとします：

```cpp
int factorial(int number) { return number <= 1 ? number : factorial(number - 1) * number; }
```

doctest を使って自己登録型テストを含むコンパイル可能な完全な例は次のようになります：

```cpp
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include "doctest.h"

int factorial(int number) { return number <= 1 ? number : factorial(number - 1) * number; }

TEST_CASE("factorial 関数のテスト") {
    CHECK(factorial(1) == 1);
    CHECK(factorial(2) == 2);
    CHECK(factorial(3) == 6);
    CHECK(factorial(10) == 3628800);
}
```

このコードをビルドすると、コマンドライン引数に反応する実行ファイルが作成されます。引数なしで実行すると、すべてのテストケース（ここでは1つ）を実行し、失敗があれば報告し、成功数／失敗数の概要を表示し、全テスト成功時は 0、何か失敗があれば 1 を返します（CI などで「動いたかどうか」の判定に使えます）。

上記を実行するとすべてのテストが通り、見た目上問題ありません。しかし実はバグが残っています。`factorial(0) == 1` の確認が抜けているのです。以下のようにチェックを追加しましょう：

```cpp
TEST_CASE("factorial 関数のテスト") {
    CHECK(factorial(0) == 1);
    CHECK(factorial(1) == 1);
    CHECK(factorial(2) == 2);
    CHECK(factorial(3) == 6);
    CHECK(factorial(10) == 3628800);
}
```

すると以下のような失敗出力が得られます：

```
test.cpp(7) FAILED!
  CHECK( factorial(0) == 1 )
with expansion:
  CHECK( 0 == 1 )
```

ここでは `factorial(0)` の戻り値（0）が出力されており、問題が一目瞭然です。

修正した関数はこちらです：

```cpp
int factorial(int number) { return number > 1 ? factorial(number - 1) * number : 1; }
```

これですべてのテストが通るようになります。

もちろん、`int` の範囲を超えるような大きな数ではオーバーフローの問題が出てくる可能性があります。現実的にはそのようなテストケースも追加し、対処法を検討すべきです。ここでは説明を省略します。

---

## この例で何をしたか？

このシンプルな例で、**doctest** の使い方について以下のポイントが示されました：

1. `#define` と `#include` を1つずつ書くだけで、テストフレームワークが利用可能になります（`main()` の実装付き）。ただしこの `#define` は **1つのソースファイルでのみ定義**してください。テストコードが複数ファイルに分かれる場合、他のファイルでは `#include "doctest.h"` のみでOKです。専用のファイルに `#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN` と `#include "doctest.h"` だけを書いておくのが良いです。また、`main()` を自前で用意して制御したい場合は[こちらを参照](main.md)してください。

2. テストケースは `TEST_CASE` マクロで記述します。引数は任意の説明的な文字列です（詳細は[テストケースとサブケース](testcases.md)参照）。名前の重複も可能で、ワイルドカードやタグで実行対象を絞り込むことができます。

3. テスト名は単なる文字列であり、関数やクラスを宣言する必要はありません。内部的には自動で関数が生成され、静的レジストリに登録されます。これによりテスト関数名の制約から解放されます。

4. アサーションは `CHECK()` マクロで記述します。比較や条件判定は C++ の自然な構文で書けるため、わざわざ `CHECK_EQ()` などの別マクロを使う必要がありません。内部では式テンプレートにより左右の値が保持され、失敗時にその値を出力してくれます。他にも多くの[アサーションマクロ](assertions.md)があります。

---

## テストケースとサブケース

多くのテストフレームワークでは、クラスベースのフィクスチャを使って `setup()` / `teardown()` を実装します（C++ ではコンストラクタ／デストラクタを利用することも）。  
**doctest** もこの形式をサポートしますが、より柔軟な仕組みとして「サブケース（`SUBCASE`）」を提供しています。

以下のコードをご覧ください：

```cpp
TEST_CASE("vector はサイズ変更可能") {
    std::vector<int> v(5);

    REQUIRE(v.size() == 5);
    REQUIRE(v.capacity() >= 5);

    SUBCASE("push_back するとサイズが増える") {
        v.push_back(1);
        CHECK(v.size() == 6);
        CHECK(v.capacity() >= 6);
    }
    SUBCASE("reserve すると容量のみ増える") {
        v.reserve(6);
        CHECK(v.size() == 5);
        CHECK(v.capacity() >= 6);
    }
}
```

このように `SUBCASE()` を使うと、各サブケースのたびに `TEST_CASE()` が最初から実行されるため、テストの独立性が保たれます。  
`CHECK()` が失敗するとその後も実行されますが、`REQUIRE()` は失敗時点でテストを中断します。

---

### サブケースのネスト（入れ子構造）

サブケースは任意に深くネスト可能です。以下の例を見てください：

#### コード（一部省略）：

```cpp
TEST_CASE("多段サブケース") {
    cout << endl << "root" << endl;
    SUBCASE("") {
        cout << "1" << endl;
        SUBCASE("") { cout << "1.1" << endl; }
    }
    SUBCASE("") {
        cout << "2" << endl;
        SUBCASE("") { cout << "2.1" << endl; }
        SUBCASE("") {
            cout << "2.2" << endl;
            SUBCASE("") {
                cout << "2.2.1" << endl;
                SUBCASE("") { cout << "2.2.1.1" << endl; }
                SUBCASE("") { cout << "2.2.1.2" << endl; }
            }
        }
        SUBCASE("") { cout << "2.3" << endl; }
        SUBCASE("") { cout << "2.4" << endl; }
    }
}
```

#### 出力：

```
root
1
1.1
root
2
2.1
root
2
2.2
2.2.1
2.2.1.1
root
2
2.2
2.2.1
2.2.1.2
root
2
2.3
root
2
2.4
```

各サブケースは完全に独立した経路で実行されます。他のケースの影響を受けません。  
また、doctest は [**スレッドセーフ**](faq.md#is-doctest-thread-aware) ですが、サブケースはメインスレッドでのみ使用し、スレッド内で使用する場合は終了前に必ず `join()` してください。

---

## スケールアップするには？

このチュートリアルではすべて1ファイルにまとめましたが、現実的な開発ではファイル分割が必要です。

以下のブロックは**1つのソースファイルだけ**に記述してください：

```cpp
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include "doctest.h"
```

他のすべてのテストファイルは単に：

```cpp
#include "doctest.h"
```

を記述すれば十分です。多くの場合、`#define` を書くファイルは専用の `.cpp` に切り出すのが良い設計です。

---

## 次のステップ

このチュートリアルは、**doctest** の概要と他のフレームワークとの違いを理解し、テストを書き始めるための導入です。

さらに深く知りたい方は、[**リファレンスセクション**](readme.md#reference)をご覧ください。

---

[ホームに戻る](readme.md#reference)