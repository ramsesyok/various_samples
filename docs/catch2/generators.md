# データジェネレータ（Data Generators）

> Catch 2.6.0 で導入されました。

データジェネレータ（_データ駆動型／パラメータ化テストケース_ とも呼ばれる）は、異なる入力値に対して同じアサーションを繰り返し使うための仕組みです。

Catch2 では `TEST_CASE` や `SECTION` マクロの順序とネスト構造を保持しつつ、  
各ジェネレータの値に対してセクションを繰り返し実行します。

以下はその基本的な例です：

```cpp
TEST_CASE("Generators") {
    auto i = GENERATE(1, 3, 5);
    REQUIRE(is_odd(i));
}
```

このテストケース "Generators" は、3回実行され、それぞれ `i` の値は 1、3、5 になります。

同じスコープで `GENERATE` を複数使うと、すべての組み合わせ（直積＝デカルト積）が生成されます。

```cpp
TEST_CASE("Generators") {
    auto i = GENERATE(1, 2);
    auto j = GENERATE(3, 4, 5);
}
```

この場合、テストケースは 2×3＝6 回実行されます。

Catch2 のジェネレータ機能には以下の2つの要素があります：

- `GENERATE` マクロと、あらかじめ用意された標準ジェネレータ群
- 独自ジェネレータを実装できる `IGenerator<T>` インターフェース

---

## `GENERATE` と `SECTION` の組み合わせ

`GENERATE` は暗黙の `SECTION` のように動作します。  
ジェネレータの位置からスコープの末尾までが対象範囲となります。

### 例：

```cpp
TEST_CASE("Generators") {
    auto i = GENERATE(1, 2);
    SECTION("one") {
        auto j = GENERATE(-3, -2);
        REQUIRE(j < i);
    }
    SECTION("two") {
        auto k = GENERATE(4, 5, 6);
        REQUIRE(i != k);
    }
}
```

- `SECTION("one")` は 2×2＝4 回実行されます  
- `SECTION("two")` は 2×3＝6 回実行されます

セクションの順序は `"one", "one", "two", "two", "two", "one"...` のように繰り返されます。

### 中間に `GENERATE` を置いた場合：

```cpp
TEST_CASE("GENERATE between SECTIONs") {
    SECTION("first") { REQUIRE(true); }
    auto _ = GENERATE(1, 2);
    SECTION("second") { REQUIRE(true); }
}
```

この例では `"first"` セクションは1回、`"second"` セクションは2回実行され、アサーションは合計3回記録されます。

```cpp
TEST_CASE("複雑なセクションとジェネレータの組み合わせ") {
    auto i = GENERATE(1, 2);
    SECTION("A") { SUCCEED("A"); }
    auto j = GENERATE(3, 4);
    SECTION("B") { SUCCEED("B"); }
    auto k = GENERATE(5, 6);
    SUCCEED();
}
```

この例では、アサーションが14回発生します。

> `GENERATE` を `SECTION` の間に配置できる機能は [Catch 2.13.0](https://github.com/catchorg/Catch2/issues/1938) で導入されました。

---

## 標準ジェネレータの種類

Catch2 の標準ジェネレータは以下のように分類されます：

### 1. 基本ジェネレータ
- `SingleValueGenerator<T>`（1要素のみ）
- `FixedValuesGenerator<T>`（複数要素）

### 2. ジェネレータ修飾子
- `FilterGenerator<T, Predicate>`：条件を満たす要素のみ通す
- `TakeGenerator<T>`：最初の N 個を取得
- `RepeatGenerator<T>`：ジェネレータの出力を N 回繰り返す
- `MapGenerator<T, U, Func>`：別のジェネレータの値に関数を適用
- `ChunkGenerator<T>`：N 個ずつに分割（`std::vector`）

### 3. 特定用途ジェネレータ
- `RandomIntegerGenerator<Integral>`：範囲内の整数乱数を生成
- `RandomFloatGenerator<Float>`：範囲内の浮動小数乱数を生成
- `RangeGenerator<T>`：等差数列を生成
- `IteratorGenerator<T>`：任意のイテレータ範囲からコピー

> `ChunkGenerator` ～ `RangeGenerator` は Catch 2.7.0 で導入  
> `IteratorGenerator` は Catch 2.10.0 で導入  
> 浮動小数での `range()` は Catch 2.11.0 で導入  

---

## ジェネレータの補助関数

より簡潔に使えるように、型推論を伴う補助関数が用意されています：

```cpp
value(42)
values({1, 2, 3})
table<std::tuple<int, std::string>>({{1, "a"}, {2, "b"}})
filter([](int x) { return x % 2 == 0; }, ...)
take(10, ...)
repeat(3, ...)
map([](int x) { return x * 2; }, ...)
chunk(4, ...)
random(0, 100)
range(0, 10)
range(0.0, 1.0, 0.1)
from_range(v.begin(), v.end())
from_range(std::vector<int>{1,2,3})
```

### 応用例：

```cpp
TEST_CASE("ランダムな奇数を生成", "[example][generator]") {
    auto i = GENERATE(take(100,
                filter([](int i) { return i % 2 == 1; },
                random(-100, 100))));
    REQUIRE(i > -100);
    REQUIRE(i < 100);
    REQUIRE(i % 2 == 1);
}
```

---

## `GENERATE_COPY` と `GENERATE_REF`

通常の `GENERATE(...)` では **外部の変数を使うことはできません**（スコープ外に生き残るため危険です）。  
変数を使いたい場合は、安全性に注意した上で `GENERATE_COPY` または `GENERATE_REF` を使用してください。

```cpp
int a = 42;
auto x = GENERATE_COPY(a + 1, a + 2);
```

> `GENERATE_COPY` / `GENERATE_REF` は Catch 2.7.1 で導入されました。

---

## 型指定による明示的な変換

たとえば文字列リテラルを `std::string` 型として扱いたい場合は、以下のようにします：

```cpp
auto str = GENERATE(as<std::string>{}, "a", "bb", "ccc");
REQUIRE(str.size() > 0);
```

---

## ジェネレータの実装（カスタム）

独自ジェネレータを実装するには、`IGenerator<T>` を継承します：

```cpp
template<typename T>
struct IGenerator : GeneratorUntypedBase {
    virtual bool next() = 0;     // 次の要素に進む。まだあれば true。
    virtual T const& get() const = 0; // 現在の要素を返す。
};
```

このままでは `GENERATE()` で使えないため、  
`std::unique_ptr<IGenerator<T>>` を `GeneratorWrapper<T>` でラップする必要があります。

---

詳しくは Catch2 の公式例：  
📂 [`examples/300-Gen-OwnGenerator.cpp`](../examples/300-Gen-OwnGenerator.cpp) を参照してください。

---

[ホームへ戻る](Readme.md)
