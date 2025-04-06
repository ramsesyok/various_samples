# アサーションマクロ（Assertion macros）

多くのテストフレームワークでは、すべての条件形式に対応するために大量のアサーションマクロ（`_EQUALS`, `_NOTEQUALS`, `_GREATER_THAN` など）を用意しています。

**doctest** はその点で異なります（ただしこの点では [**Catch**](https://github.com/catchorg/Catch2) と似ています）。doctest は比較式を分解して評価するため、ほとんどのケースで 1〜2 種類のマクロだけで十分になります。それでも補助的なマクロは豊富に用意されています。

---

## アサーションの重大度レベル

すべてのアサーションマクロには、次の3つの重大度レベルがあります：

- `REQUIRE`：失敗すると**直ちにテストケースを終了**し、テストケースを失敗とマークします。
- `CHECK`：失敗しても**テストを継続**し、テストケースは失敗としてマークされます。
- `WARN`：失敗しても**テストは失敗とみなされず**、メッセージだけが出力されます。

`CHECK` は、互いに独立した複数のアサーションをすべて評価したいときに便利です。

すべてのアサーションは式を一度だけ評価し、失敗した場合は[**文字列化（stringification）**](stringification.md)されて詳細に表示されます。

doctest は [**スレッドセーフ**](faq.md#is-doctest-thread-aware) なため、スレッド内でもアサーションマクロや [**ログマクロ**](logging.md) を使用できます。

> 注意：`REQUIRE` レベルのアサーションは、テストケース終了のために例外をスローします。そのため、ユーザー定義クラスのデストラクタ内で使うのは危険です。例外処理中のスタック巻き戻し中にさらに例外が発生すると、`std::terminate()` が呼ばれます。C++11以降、デストラクタはデフォルトで `noexcept(true)` です。

---

## 式分解型アサーション（Expression Decomposing）

形式は `CHECK(式)` です（`REQUIRE` や `WARN` も同様に使用可能）。

比較式（例：`a == b`）や単一の式（例：`vec.isEmpty()`）を記述できます。

例外が発生した場合はキャッチされ、失敗としてカウントされます（ただし `WARN` レベルではカウントされません）。

### 例：

```cpp
CHECK(flags == state::alive | state::moving);
CHECK(thisReturnsTrue());
REQUIRE(i < 42);
```

### 否定型アサーション（`_FALSE`）

```<LEVEL>_FALSE(expression)``` は式の結果の論理否定（`!`）を評価します。`!` を使うと式の分解ができないため、この形式が用意されています。

```cpp
REQUIRE_FALSE(thisReturnsFalse());
```

### `_MESSAGE` 付きの形式

`CHECK_MESSAGE(expr, msg)` のように、スコープ付きログメッセージを併せて表示できます。

```cpp
INFO("これはすべてのアサートに関連する情報");

CHECK_MESSAGE(a < b, "これはこのアサートだけに関連する情報: ", localVar);
```

---

## バイナリ／単項アサーション（Binary / Unary）

比較式の分解を行わない高速なマクロです。テンプレートを使わず、コンパイル時間が[**57〜68% 高速**](benchmarks.md#cost-of-an-assertion-macro)になります。

```<LEVEL>``` には `REQUIRE`、`CHECK`、`WARN` のいずれかを指定します。

### バイナリ比較：

- `<LEVEL>_EQ(a, b)` → `a == b`
- `<LEVEL>_NE(a, b)` → `a != b`
- `<LEVEL>_GT(a, b)` → `a >  b`
- `<LEVEL>_LT(a, b)` → `a <  b`
- `<LEVEL>_GE(a, b)` → `a >= b`
- `<LEVEL>_LE(a, b)` → `a <= b`

### 単項比較：

- `<LEVEL>_UNARY(expr)` → `expr`
- `<LEVEL>_UNARY_FALSE(expr)` → `!expr`

[`DOCTEST_CONFIG_SUPER_FAST_ASSERTS`](configuration.md#doctest_config_super_fast_asserts) を定義すると、バイナリ型アサーションのコンパイルが[**84〜91% 高速化**](benchmarks.md#cost-of-an-assertion-macro)されます。

---

## 例外に関するアサーション

- `<LEVEL>_THROWS(expr)`  
  任意の型の例外が発生することを期待します。

- `<LEVEL>_THROWS_AS(expr, exception_type)`  
  指定した型の例外が発生することを期待します（`const` や `&` は自動補完されます）。

```cpp
CHECK_THROWS_AS(func(), const std::exception&);
CHECK_THROWS_AS(func(), std::exception); // 上と同じ
```

- `<LEVEL>_THROWS_WITH(expr, "message")`  
  例外のメッセージが指定文字列と一致することを期待します。メッセージは [例外の文字列化](stringification.md#translating-exceptions) を参照。

```cpp
CHECK_THROWS_WITH(func(), "invalid operation!");
```

- `<LEVEL>_THROWS_WITH_AS(expr, "message", exception_type)`  
  メッセージ＋型の両方をチェックします。

```cpp
CHECK_THROWS_WITH_AS(func(), "invalid operation!", std::runtime_error);
```

- `<LEVEL>_NOTHROW(expr)`  
  式評価中に例外が**発生しないこと**を期待します。

> `_MESSAGE` 付きバージョンもあり、通常のアサーション同様にログと併用可能です。

> 複数文や関数ポインタではなく、即時実行ラムダや関数呼び出し式を渡してください。  
> 例：`[&]() { throw 1; }()`（末尾の `()` が必要）

[`DOCTEST_CONFIG_VOID_CAST_EXPRESSIONS`](configuration.md#doctest_config_void_cast_expressions) を使うと、式を `(void)` にキャストして警告を回避できます。

---

## テスト外でのアサーション使用

doctest のアサーションは、`TEST_CASE()` の外でも `assert()` の代替として使用可能です。

使用には `doctest::Context` オブジェクトを作成し、`setAsDefaultForAssertsOutOfTestCases()` を呼び出して既定コンテキストとして登録する必要があります。  
失敗時に独自のハンドラを設定するには `setAssertHandler()` を使用します。未設定の場合、失敗時には `std::abort()` が呼ばれます。

[使用例はこちら](../../examples/all_features/asserts_used_outside_of_tests.cpp)  
関連の[issueはこちら](https://github.com/doctest/doctest/issues/114)

> 現在、テスト外のアサートでは [ログマクロ](logging.md) は使えません。  
> つまり `_MESSAGE` 付きアサートも使えません（`INFO()` とアサートの組み合わせであるため）。

---

## 文字列の部分一致チェック

`doctest::Contains` を使うと、文字列が別の文字列に含まれているかをチェックできます。

```cpp
REQUIRE("foobar" == doctest::Contains("foo"));
```

これは `THROWS_WITH` 系マクロでも使えます：

```cpp
REQUIRE_THROWS_WITH(func(), doctest::Contains("Oopsie"));
```

---

## 浮動小数点の比較

浮動小数点数の比較には、**誤差**への配慮が必要です。  
**doctest** では `doctest::Approx` クラスを使って、許容誤差付きの比較が可能です。

```cpp
REQUIRE(performComputation() == doctest::Approx(2.1));
```

許容誤差（相対誤差）は `.epsilon()` で指定できます：

```cpp
REQUIRE(22.0/7 == doctest::Approx(3.141).epsilon(0.01)); // 1% 誤差許容
```

非常に大きい／小さい数値には `.scale()` を使ってスケール指定も可能です。

---

## NaN チェック

C++ の NaN は `NaN != NaN` であるため、通常の比較はできません。

```cpp
CHECK(std::isnan(performComputation())); // 戻り値はキャプチャされない
```

doctest の `doctest::IsNaN` を使えば、戻り値をキャプチャして NaN チェックできます：

```cpp
CHECK(doctest::IsNaN(performComputation())); // 戻り値も表示される
```

`!doctest::IsNaN(...)` のように否定しても使用可能です。

---

- 多くのアサーションマクロを網羅した [使用例](../../examples/all_features/assertion_macros.cpp) をご覧ください。
- `try`/`catch` でアサーションマクロを囲まないでください。`REQUIRE` マクロはテスト終了のために例外をスローします。

---

[ホームに戻る](readme.md#reference)
