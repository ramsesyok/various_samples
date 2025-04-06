# 文字列変換（String conversions）

**doctest** では、アサーションやログに使われる型を文字列に変換する必要があります（失敗時のメッセージなどのため）。  
ビルトイン型のほとんどは既に対応済みですが、ユーザー定義型やサードパーティ型については、次の3つの方法で対応可能です。

> **enum の文字列化**については[この issue](https://github.com/doctest/doctest/issues/121)を参照してください。

---

## 方法①：`std::ostream` への `operator<<` をオーバーロードする

もっとも標準的な方法です。すでに独自型でこの演算子を定義している場合は、そのまま使えます。

```cpp
std::ostream& operator<<(std::ostream& os, const T& value) {
    os << convertMyTypeToString(value); // 独自の変換関数
    return os;
}
```

この関数は、型 `T` の属する**名前空間内に定義**する必要があります。

### メンバ関数として書くことも可能ですが非推奨です：

```cpp
std::ostream& T::operator<<(std::ostream& os) const {
    os << convertMyTypeToString(*this);
    return os;
}
```

---

## 方法②：`doctest::toString()` をオーバーロードする

`operator<<` を使いたくない、あるいはテスト用に別の出力形式にしたい場合は、doctest の `toString()` をオーバーロードする方法があります：

```cpp
namespace user {
    struct udt {};

    doctest::String toString(const udt& value) {
        return convertMyTypeToString(value);
    }
}
```

- `toString()` は型と同じ名前空間に定義する必要があります。
- 型がグローバル名前空間にある場合は、関数もグローバルに。

---

## 方法③：`doctest::StringMaker<T>` を特殊化する

`toString()` が正しく動作しないケースや、より強力な制御が必要な場合は `StringMaker<T>` の特殊化が推奨されます：

```cpp
namespace doctest {
    template<> struct StringMaker<T> {
        static String convert(const T& value) {
            return convertMyTypeToString(value);
        }
    };
}
```

少し冗長になりますが、信頼性の高い方法です。

---

## 例外の文字列変換（Translating exceptions）

デフォルトでは、`std::exception` を継承した例外は `what()` を呼び出して自動で文字列に変換されます。

### 独自の例外型や `what()` では不十分な場合：

```cpp
REGISTER_EXCEPTION_TRANSLATOR(MyType& ex) {
    return doctest::String(ex.message()); // 独自の例外メッセージ取得
}
```

- 参照渡しが推奨されます。
- 登録場所はどこでもよく、複数の翻訳単位でもOKです。

### ラムダ式を使った登録も可能：

```cpp
doctest::registerExceptionTranslator<int>(
    [](int in) { return doctest::toString(in); }
);
```

登録順を制御したい場合は、1つのソースファイルでトップダウンに並べるのが確実です。

---

## 補足事項

- 例や `std::vector<T>` の stringification については [このサンプル](../../examples/all_features/stringification.cpp) を参照。
- `doctest::String` は doctest 独自の文字列型です（`std::string` は使いません）。
- `operator<<` による変換をサポートするために `std::ostream` を前方宣言しています。  
  標準的には好ましくありませんが、実際にはすべての主要コンパイラで問題なく動作しています。
- 100% 標準準拠を目指す場合は、`<iosfwd>` を強制インクルードする [`DOCTEST_CONFIG_USE_STD_HEADERS`](configuration.md#doctest_config_use_std_headers) を使ってください。

---

[ホームに戻る](readme.md#reference)
