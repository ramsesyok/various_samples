# 文字列変換

Catch は、アサーションやログ出力で使用するあらゆる型を文字列に変換できる必要があります（ログやレポート用）。  
多くの組み込み型や標準ライブラリ型はデフォルトで対応していますが、**独自の型やサードパーティの型**を Catch に対応させるには、次のいずれかの方法を使います。

---

## std::ostream の operator << オーバーロード

これは C++ における標準的な文字列化手段です。多くの場合、既に実装済みかもしれません。  
未実装の場合は、次のようにフリー関数を用意します：

```cpp
std::ostream& operator << ( std::ostream& os, T const& value ) {
    os << convertMyTypeToString( value );
    return os;
}
```

※ `T` は自分の型名、`convertMyTypeToString` は変換処理（直接ここに書いてもOK）  
この関数は **型と同じ名前空間**か、**グローバル名前空間**に置き、**Catch のインクルード前に定義**してください。

---

## Catch::StringMaker の特殊化

`operator<<` を定義したくない場合や、**テスト用に別の文字列変換をしたい場合**は、Catch の `StringMaker` を特殊化できます。

```cpp
namespace Catch {
    template<>
    struct StringMaker<T> {
        static std::string convert( T const& value ) {
            return convertMyTypeToString( value );
        }
    };
}
```

---

## Catch::is_range の特殊化

Catch は、`begin(T)` と `end(T)` が有効な型を「イテレータで扱える範囲型」とみなし、要素を列挙するように文字列化します。  
ただし、型によってはこれにより **無限再帰**が発生する可能性があるため、無効化も可能です。

```cpp
namespace Catch {
    template<>
    struct is_range<T> {
        static const bool value = false;
    };
}
```

---

## 例外

デフォルトでは、`std::exception` を継承した例外は `what()` メソッドの結果で文字列化されます。  
それ以外の型や `what()` の内容が不十分な場合は、`CATCH_TRANSLATE_EXCEPTION` マクロを使って変換関数を定義できます：

```cpp
CATCH_TRANSLATE_EXCEPTION( MyType& ex ) {
    return ex.message();
}
```

※ これはどのファイルに書いても構いません。

---

## 列挙型（enum）

> Catch 2.8.0 で導入されました。

すでに `operator<<` が定義されている列挙型はそのまま文字列化されます。  
それ以外は、`StringMaker` を特殊化するか、便利なマクロ `CATCH_REGISTER_ENUM` を使うことで簡単に対応できます。

### 例 1: 通常の enum

```cpp
enum class Fruits { Banana, Apple, Mango };

CATCH_REGISTER_ENUM( Fruits, Fruits::Banana, Fruits::Apple, Fruits::Mango )

TEST_CASE() {
    REQUIRE( Fruits::Mango == Fruits::Apple );
}
```

### 例 2: 名前空間つきの enum

```cpp
namespace Bikeshed {
    enum class Colours { Red, Green, Blue };
}

// マクロは名前空間の外（トップレベル）に書く必要があります
CATCH_REGISTER_ENUM( Bikeshed::Colours,
    Bikeshed::Colours::Red,
    Bikeshed::Colours::Green,
    Bikeshed::Colours::Blue )

TEST_CASE() {
    REQUIRE( Bikeshed::Colours::Red == Bikeshed::Colours::Blue );
}
```

---

## 浮動小数点数の精度

> [Catch 2.8.0](https://github.com/catchorg/Catch2/issues/1614) で導入されました。

`float` と `double` には `StringMaker` の既定実装があります。  
デフォルトの精度は一般的な用途に適していますが、次のように精度を変更できます：

```cpp
Catch::StringMaker<float>::precision = 15;
const float testFloat1 = 1.12345678901234567899f;
const float testFloat2 = 1.12345678991234567899f;
REQUIRE(testFloat1 == testFloat2);
```

上記のアサーションは失敗し、**15桁の精度**で `testFloat1` と `testFloat2` が出力されます。

---

[ホームに戻る](Readme.md)
