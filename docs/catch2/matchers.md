# マッチャー（Matchers）

マッチャーは、アサーションを記述するための代替手段であり、簡単に拡張・組み合わせができます。  
そのため、コレクションのような複雑な型や、自作の型と組み合わせるのに非常に適しています。

マッチャーは [Hamcrest](https://en.wikipedia.org/wiki/Hamcrest) ファミリーのフレームワークによって広く知られるようになりました。

---

## 使い方

マッチャーは `REQUIRE_THAT` または `CHECK_THAT` マクロで使用します。これらは2つの引数を取ります：

- 最初の引数：テスト対象のオブジェクトや値  
- 2つ目の引数：マッチ _式_（単一のマッチャー、または `&&`、`||`、`!` を使って組み合わせたもの）

たとえば、文字列が特定の語句で終わることを確認するには：

```c++
using Catch::Matchers::EndsWith; // または Catch::EndsWith
std::string str = getStringFromSomewhere();
REQUIRE_THAT( str, EndsWith( "as a service" ) );
```

マッチャーは複数の引数を取ることができ、より詳細な指定が可能です。  
たとえば、文字列マッチャーには大文字小文字を区別するかどうかの指定ができます：

```c++
REQUIRE_THAT( str, EndsWith( "as a service", Catch::CaseSensitive::No ) );
```

マッチャーは以下のように組み合わせることも可能です：

```c++
REQUIRE_THAT( str,
    EndsWith( "as a service" ) ||
    (StartsWith( "Big data" ) && !Contains( "web scale" ) ) );
```

⚠️ `&&` や `||` で結合したマッチャー式はオブジェクトの所有権を持ちません。  
そのため、以下のようにローカル変数に代入すると、ライフタイムの問題により use-after-free（解放後使用）が発生し、クラッシュする可能性があります：

```cpp
TEST_CASE("Bugs, bugs, bugs", "[Bug]"){
    std::string str = "Bugs as a service";

    auto match_expression = Catch::EndsWith( "as a service" ) ||
        (Catch::StartsWith( "Big data" ) && !Catch::Contains( "web scale" ) );
    REQUIRE_THAT(str, match_expression);
}
```

---

## 組み込みマッチャー

Catch2 にはいくつかのマッチャーが標準で用意されています。  
これらは `Catch::Matchers::foo` 名前空間に定義されており、`Catch` 名前空間にもインポートされています。

各マッチャーには、テンプレート引数を自動推論するための「ヘルパー関数」も用意されています。  
たとえば、`std::vector` が完全に一致するかをチェックするマッチャーは `EqualsMatcher<T>` ですが、通常は `Equals` 関数を使います。

---

### 文字列マッチャー

以下のマッチャーが用意されています：

- `StartsWith`  
- `EndsWith`  
- `Contains`  
- `Equals`  
- `Matches`（ECMAScript の正規表現による完全一致）

`Matches` は文字列全体にマッチする必要があるため、たとえば `"abc"` は `"abcd"` にマッチしません。正規表現 `"abc.*"` ならマッチします。

すべての文字列マッチャーは、オプションで大文字小文字を区別するかどうかを指定できます（デフォルトでは区別あり）。

---

### ベクタマッチャー（`std::vector`）

Catch2 は以下の5つの `std::vector` 用マッチャーを提供しています：

- `Contains`：指定したベクタが含まれているか
- `VectorContains`：指定した要素が含まれているか
- `Equals`：ベクタが完全に一致しているか（順序も一致）
- `UnorderedEquals`：順序は無視して要素が一致しているか
- `Approx`：`Approx` を用いた近似一致（順序あり）

> `Approx` マッチャーは [Catch 2.7.2 で導入](https://github.com/catchorg/Catch2/issues/1499) されました。

---

### 浮動小数点マッチャー

Catch2 には以下の3つの浮動小数点用マッチャーがあります：

- `WithinAbsMatcher`：絶対誤差以内か  
  → `WithinAbs(target, margin)` で構築

- `WithinUlpsMatcher`：ULP（最下位ビット）単位での比較  
  → `WithinULP(float, int64_t)` または `WithinULP(double, int64_t)`

- `WithinRelMatcher`：相対誤差での近似一致  
  → `WithinRel(target, margin)` または `WithinRel(target)`（デフォルトは machine epsilon × 100）

ULP に関する詳細は [Wikipedia: Unit in the Last Place](https://en.wikipedia.org/wiki/Unit_in_the_last_place) を参照してください。

> `WithinRel` は Catch 2.10.0 で導入されました。

---

### 汎用マッチャー

Catch2 は汎用マッチャーも提供しています。  
現在は、任意の述語（関数オブジェクト）を使えるマッチャーが含まれています。

ただし、型推論の制限により、述語の引数型は明示的に指定する必要があります：

```cpp
REQUIRE_THAT("Hello olleH",
    Predicate<std::string>(
        [] (std::string const& str) -> bool { return str.front() == str.back(); },
        "First and last character should be equal" // 最初と最後の文字が一致すべき
    )
);
```

第2引数は任意の説明文で、テストレポートに表示されます。

---

### 例外マッチャー

Catch2 では、例外のメッセージ文字列が完全一致するかを検証する例外マッチャーも用意されています。

- マッチャークラス名：`ExceptionMessageMatcher`
- ヘルパー関数：`Message`

このマッチャーを使うには、対象の例外が `std::exception` を継承している必要があります。  
また、メッセージの一致は大文字小文字も含めて完全一致です。

> `ExceptionMessageMatcher` は Catch 2.10.0 で導入されました。

使用例：

```cpp
REQUIRE_THROWS_MATCHES(throwsDerivedException(), DerivedException, Message("DerivedException::what"));
```

---

## カスタムマッチャーの作成

Catch を拡張したり、自作の型に対してマッチャーを使いたい場合は、簡単にカスタムマッチャーを作れます。

必要なのは以下の2つです：

1. `Catch::MatcherBase<T>` を継承したマッチャークラス（`T` は対象型）  
   - コンストラクタで比較対象などを保持  
   - `match()` と `describe()` をオーバーライド

2. ビルダ関数（実際のテストコードで呼び出すための関数）

以下は、整数が特定の範囲内にあるかを検証するマッチャーの例です：

```cpp
// マッチャークラス
class IntRange : public Catch::MatcherBase<int> {
    int m_begin, m_end;
public:
    IntRange( int begin, int end ) : m_begin( begin ), m_end( end ) {}

    // 実際のテスト処理を行う
    bool match( int const& i ) const override {
        return i >= m_begin && i <= m_end;
    }

    // マッチャーの説明文を生成
    virtual std::string describe() const override {
        std::ostringstream ss;
        ss << "is between " << m_begin << " and " << m_end;
        return ss.str();
    }
};

// ビルダ関数
inline IntRange IsBetween( int begin, int end ) {
    return IntRange( begin, end );
}

// 使用例
TEST_CASE("Integers are within a range")
{
    CHECK_THAT( 3, IsBetween( 1, 10 ) );
    CHECK_THAT( 100, IsBetween( 1, 10 ) );
}
```

このテストを実行すると、次のような出力が得られます：

```cpp
/**/TestFile.cpp:123: FAILED:
  CHECK_THAT( 100, IsBetween( 1, 10 ) )
with expansion:
  100 is between 1 and 10
```

---

[ホームへ戻る](Readme.md)
