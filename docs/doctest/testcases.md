# テストケース（Test cases）

**doctest** は、従来の xUnit スタイル（クラスベースのフィクスチャにテストメソッドを定義するスタイル）を完全にサポートしていますが、推奨スタイルではありません。代わりに **doctest** は、テストケース内にサブケース（`SUBCASE`）をネストできる強力な仕組みを提供しています。詳細や具体例については[チュートリアルの該当セクション](tutorial.md#test-cases-and-subcases)を参照してください。

テストケースとサブケースの基本構文はとても簡単です：

- `TEST_CASE("テスト名")`
- `SUBCASE("サブケース名")`

`テスト名` や `サブケース名` は自由な形式の文字列リテラルです。doctest ではテスト名は一意である必要はありません。

C++17 以降では、`TEST_CASE_CLASS()` を使うことでクラス本体の中にテストケースを書くことができ、プライベートメンバーのテストがしやすくなります。

> 注意：**doctest** は[スレッドセーフ](faq.md#is-doctest-thread-aware)ですが、サブケースは**メインスレッド内**でのみ使用してください。

---

## パラメータ化テスト（Parameterized test）

doctest はテストケースのパラメータ化もサポートしています。詳細は[こちらのドキュメント](parameterized-tests.md)をご覧ください。

---

## コマンドラインでのフィルタリング

テストケースやサブケースは、[コマンドラインオプション](commandline.md)で名前やタグに基づいて絞り込み実行が可能です。

---

## BDD スタイルのテストケース（シナリオ記法）

**doctest** は BDD（振る舞い駆動開発）の理念に沿った記法もサポートしています。これは「実行可能な仕様」としてテストを書くことを目指すスタイルで、`TEST_CASE` や `SUBCASE` をラップしたマクロで実現されています。

- `SCENARIO("シナリオ名")`  
  → `TEST_CASE` と同じですが、名前に `"Scenario: "` が自動で付きます。

- `SCENARIO_TEMPLATE("名前", 型, 型リスト)`  
  → `TEST_CASE_TEMPLATE` と同等。

- `SCENARIO_TEMPLATE_DEFINE("名前", 型, ID)`  
  → `TEST_CASE_TEMPLATE_DEFINE` と同等。

- `GIVEN("前提条件")`  
- `WHEN("実行条件")`  
- `THEN("期待結果")`  
- `AND_WHEN("さらに実行条件")`  
- `AND_THEN("さらに期待結果")`  

これらはすべて `SUBCASE` と同様に動作しますが、それぞれ名前に `"given: "`, `"when: "`, `"then: "` のようなプレフィックスが追加されます。  
これにより、レポーター（標準出力など）で出力が整形され、可読性が向上します。

> **注意**：`--test-case=Scenario:*` のようにフィルタする際は `"Scenario: "` のプレフィックスも指定してください。

---

## テストフィクスチャ（Test fixtures）

doctest では、サブケースによるテストグルーピングに加えて、従来通りの「テストフィクスチャ」も使用できます。  
フィクスチャは通常のクラスとして定義し、`TEST_CASE_FIXTURE` マクロで使用します。

### 例：

```cpp
class UniqueTestsFixture {
private:
    static int uniqueID;
protected:
    DBConnection conn;
public:
    UniqueTestsFixture() : conn(DBConnection::createConnection("myDB")) {}
protected:
    int getID() {
        return ++uniqueID;
    }
};
int UniqueTestsFixture::uniqueID = 0;

TEST_CASE_FIXTURE(UniqueTestsFixture, "名前なしで社員作成") {
    REQUIRE_THROWS(conn.executeSQL("INSERT INTO employee (id, name) VALUES (?, ?)", getID(), ""));
}
TEST_CASE_FIXTURE(UniqueTestsFixture, "通常の社員作成") {
    REQUIRE(conn.executeSQL("INSERT INTO employee (id, name) VALUES (?, ?)", getID(), "Joe Bloggs"));
}
```

このように、共通のセットアップ／メンバ変数／ヘルパーメソッドを持つフィクスチャを使ってテストを整理できます。

---

## テストスイート（Test suites）

テストケースは「テストスイート」としてグループ化できます：

```cpp
TEST_CASE("") {} // どのスイートにも属さない

TEST_SUITE("math") {
    TEST_CASE("") {} // math スイートに属する
    TEST_CASE("") {}
}

TEST_SUITE_BEGIN("utils");

TEST_CASE("") {} // utils スイートに属する

TEST_SUITE_END();

TEST_CASE("") {} // スイート外
```

テストスイート単位で実行するには、コマンドラインでフィルタを使います（例：`--test-suite=math`）。詳細は[コマンドラインの項目](commandline.md)を参照してください。

---

## デコレータ（Decorators）

テストケースに追加の属性（メタデータ）を付加できます：

```cpp
TEST_CASE("name"
          * doctest::description("500ms以内に終わるべき")
          * doctest::timeout(0.5)) {
    // アサーション
}
```

複数のデコレータを組み合わせて使用可能です。以下のようなものがあります：

- `skip(bool)`：テストをスキップ（`--no-skip` を指定すれば実行される）
- `no_breaks(bool)`：アサート失敗時にデバッガでブレークしない
- `no_output(bool)`：アサートの出力を抑制
- `may_fail(bool)`：失敗してもテストは失敗としない（作業中の機能など）
- `should_fail(bool)`：成功した場合に失敗とする（バグ修正漏れの検出など）
- `expected_failures(int)`：失敗するアサートの想定数（異なる場合は失敗と判定）
- `timeout(double)`：テスト実行時間の上限（秒）。強制終了はしない
- `test_suite("name")`：テストケースの属するスイートを設定・上書き
- `description("text")`：テスト内容の説明

> これらは `main()` の前（グローバル初期化時）に評価・登録されます。

### テストスイートへの一括適用

```cpp
TEST_SUITE("some TS" * doctest::description("全テストに説明を適用")) {
    TEST_CASE("継承された説明付きテスト") {
        // asserts
    }
}
TEST_SUITE("some TS") {
    TEST_CASE("説明なしのテスト") {
        // asserts
    }
}
```

### テストケース側でデコレータを上書き

```cpp
TEST_SUITE("タイムアウト500ms" * doctest::timeout(0.5)) {
    TEST_CASE("500ms制限") {
        // asserts
    }
    TEST_CASE("200ms制限" * doctest::timeout(0.2)) {
        // asserts
    }
}
```

---

- [サブケースと BDD 記法のサンプル](../../examples/all_features/subcases.cpp)
- [アサーションマクロのサンプル](../../examples/all_features/assertion_macros.cpp)
- テストは各 cpp ファイル内で上から順に登録されますが、cpp 間の順序には保証がありません。

---

[ホームに戻る](readme.md#reference)
