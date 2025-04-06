# テストケースとセクション

Catch は、従来の xUnit スタイル ― クラスベースのフィクスチャにテストメソッドを記述するスタイル ― に完全に対応していますが、それは推奨されるスタイルではありません。

Catch では、テストケース内にセクションをネストできる強力な仕組みが提供されています。詳しくは [チュートリアル](tutorial.md#test-cases-and-sections) を参照してください。

テストケースとセクションの構文は非常にシンプルです：

* **TEST_CASE(** _テスト名_ \[, _タグ_ \] **)**
* **SECTION(** _セクション名_ **)**

_テスト名_ や _セクション名_ はクォートで囲まれた自由な文字列です。オプションの _タグ_ 引数は、角括弧で囲まれた1つ以上のタグからなる文字列です。タグについては以下で解説します。テスト名は Catch 実行ファイル内で一意でなければなりません。

例は [チュートリアル](tutorial.md) を参照してください。

---

## タグ

タグを使うと、任意の文字列をテストケースに付加できます。タグを使って、特定のテストケースを選択して実行・表示することができます。複数のタグを論理式のように組み合わせることも可能です。タグは、関連する複数のテストをグループ化するのに便利です。

以下のテストケースがあるとします：

```cpp
TEST_CASE( "A", "[widget]" ) { /* ... */ }
TEST_CASE( "B", "[widget]" ) { /* ... */ }
TEST_CASE( "C", "[gadget]" ) { /* ... */ }
TEST_CASE( "D", "[widget][gadget]" ) { /* ... */ }
```

- タグ式 `"[widget]"` → A, B, D が選択されます  
- `"[gadget]"` → C, D  
- `"[widget][gadget]"` → D のみ  
- `"[widget],[gadget]"` → すべて（A, B, C, D）

コマンドラインでの選択方法については [コマンドラインドキュメント](command-line.md#specifying-which-tests-to-run) を参照してください。

タグは大文字・小文字を区別しません。ASCII 文字ならスペースや記号も使用可能です。例：`[tag with spaces]` や `[I said "good day"]`。ただしエスケープシーケンスはサポートされておらず、`[\]]` は無効です。

---

### 特殊タグ

非英数字で始まるタグ名は Catch に予約されています。以下は定義済みの特殊タグです：

- `[!hide]` または `[.]`  
  → 通常の実行（明示的に指定されていない）から除外。`[.integration]` のように独自タグと併用するのが一般的。

- `[!throws]`  
  → 正常なテストでも例外を投げる可能性があることを明示します。`--nothrow` オプション使用時に除外されます。

- `[!mayfail]`  
  → アサーションが失敗してもテスト自体は失敗と見なされません。進行中の機能や既知の問題の追跡に便利です。

- `[!shouldfail]`  
  → `[!mayfail]` に似ていますが、逆に成功すると失敗扱いになります。修正されたバグを検出したいときに便利です。

- `[!nonportable]`  
  → 実行環境（プラットフォームやコンパイラ）によって動作が異なる可能性があることを示します。

- `[#<ファイル名>]`  
  → `--filenames-as-tags` オプションで、すべてのテストにファイル名（拡張子なし）をタグとして追加。

- `[@<エイリアス名>]`  
  → タグエイリアスの定義。次節で説明。

- `[!benchmark]`  
  → このテストケースはベンチマークです（未ドキュメント機能）。詳細は `projects/SelfTest/Benchmark.tests.cpp` を参照。

---

## タグエイリアス

複雑なタグ式やワイルドカード名を組み合わせてテスト実行パターンを作ることができます。  
このような式を頻繁に使う場合、エイリアスを定義すると便利です。

構文：
```cpp
CATCH_REGISTER_TAG_ALIAS( <エイリアス文字列>, <タグ式> )
```

例：
```cpp
CATCH_REGISTER_TAG_ALIAS( "[@nhf]", "[failing]~[.]" )
```

これにより、コマンドラインで `[@nhf]` を指定すると `[failing]` タグを持ち、`[.]` を持たないテストが実行されます。

---

## BDDスタイルのテストケース

Catch では、従来のテストケースに加えて、BDD（振る舞い駆動開発）スタイルの記述も可能です。  
BDD スタイルでは「実行可能な仕様」としてテストを記述できます。  
これらのマクロは `TEST_CASE` や `SECTION` にマッピングされ、より読みやすく整形されます。

- **SCENARIO(** _シナリオ名_ \[, _タグ_ \] **)**  
  → `TEST_CASE` に対応。名前の前に `"Scenario: "` が付きます。

- **GIVEN(** _前提条件_ **)**  
- **WHEN(** _操作_ **)**  
- **THEN(** _期待結果_ **)**  
  → 各 `SECTION` に対応し、ラベルの前に `"given: "`, `"when: "`, `"then: "` が付きます。

- **AND_GIVEN**, **AND_WHEN**, **AND_THEN**  
  → 複数の `GIVEN`、`WHEN`、`THEN` をつなげる際に使用（前に `"and "` が付きます）

> `AND_GIVEN` は [Catch 2.4.0 で導入](https://github.com/catchorg/Catch2/issues/1360) されました。

これらのマクロを使用すると、コンソール出力時に整形され、読みやすくなります。  
ただし、記述順序や正しさの強制はされないため、使用方法は開発者に委ねられます。

---

## 型パラメータ付きテストケース { #type-parametrised-test-cases}

Catch2 では `TEST_CASE` に加えて、型ごとにパラメータ化されたテストも可能です：

### TEMPLATE_TEST_CASE

```cpp
TEMPLATE_TEST_CASE( "テスト名", "[タグ]", int, std::string, (std::tuple<int, float>) )
```

> [Catch 2.5.0 で導入](https://github.com/catchorg/Catch2/issues/1437)

- タグは必須ですが空でも構いません。
- `TestType` という名前で現在の型が利用可能です。
- 複数テンプレート引数の型は `()` で囲んでください。

```cpp
std::vector<TestType> v(5);
```

---

### TEMPLATE_PRODUCT_TEST_CASE

テンプレートテンプレート型と引数の組み合わせを用いたテストです。

```cpp
TEMPLATE_PRODUCT_TEST_CASE("説明", "[タグ]", (std::vector, Foo), (int, float))
```

- 組み合わせの総数は「テンプレート型 × 引数の数」になります。
- `TestType` で結果型が利用可能です。

多引数型は `(int, float)` のように括弧で囲みます。

---

### TEMPLATE_LIST_TEST_CASE

```cpp
TEMPLATE_LIST_TEST_CASE("説明", "[タグ]", 型リスト)
```

> [Catch 2.9.0 で導入](https://github.com/catchorg/Catch2/issues/1627)

- 型リストとして `std::tuple`, `boost::mpl::list`, `boost::mp11::mp_list` などが利用可能。
- 複数のテストで同じ型リストを使い回すのに便利です。

---

## シグネチャベースのパラメータ付きテストケース { #signature-based-parametrised-test-cases }

> [Catch 2.8.0 で導入](https://github.com/catchorg/Catch2/issues/1609)

NTTP（非型テンプレート引数）なども含めたテストが可能です。

### TEMPLATE_TEST_CASE_SIG

```cpp
TEMPLATE_TEST_CASE_SIG("説明", "[タグ]", ((typename T, int V), T, V), (int,5), ...)
```

- `T`, `V` のようにテンプレート引数に名前を付けて使用可能
- `std::array<T, V>` のような型に有用

---

### TEMPLATE_PRODUCT_TEST_CASE_SIG

```cpp
TEMPLATE_PRODUCT_TEST_CASE_SIG("説明", "[タグ]", ((typename T, size_t S), T, S), (テンプレート型...), (引数...))
```

例：
```cpp
template<typename T, size_t S>
struct Bar {
    size_t size() { return S; }
};
```

```cpp
TEMPLATE_PRODUCT_TEST_CASE_SIG("説明", "[タグ]", ((typename T, size_t S), T, S), (std::array, Bar), ((int, 9), (float, 42)))
```

---

[ホームへ戻る](Readme.md)
