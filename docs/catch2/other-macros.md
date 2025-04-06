# その他のマクロ

このページでは、他のセクションで解説されていないマクロをまとめて解説します。  
現時点では、以下の2つのカテゴリに大別されています：

- **アサーション関連のマクロ**
- **テストケース関連のマクロ**

---

## アサーション関連のマクロ

### `CHECKED_IF` / `CHECKED_ELSE`

`CHECKED_IF(expr)` は `if` の代替であり、Catch2 の文字列化機構を使って `expr` を評価・記録します。  
`CHECKED_IF` の後のブロックは、`expr` が `true` のときのみ実行されます。

`CHECKED_ELSE(expr)` は `expr` が `false` のときにブロックが実行されます。

#### 例：
```cpp
int a = ...;
int b = ...;
CHECKED_IF( a == b ) {
    // a == b の場合に実行
} CHECKED_ELSE( a == b ) {
    // a != b の場合に実行
}
```

---

### `CHECK_NOFAIL`

`CHECK_NOFAIL(expr)` は `CHECK(expr)` の変種で、`false` でもテストケース全体は失敗としません。  
前提条件などが破られていても、テストを中断せずに情報として記録したい場合に便利です。

#### 出力例：
```
main.cpp:6:
FAILED - but was ok:
  CHECK_NOFAIL( 1 == 2 )

main.cpp:7:
PASSED:
  CHECK( 2 == 2 )
```

---

### `SUCCEED`

`SUCCEED(msg)` は実質的に `INFO(msg); REQUIRE(true);` と等価です。  
特定の場所まで到達すれば成功とみなすような場面で使います。

#### 例：
```cpp
TEST_CASE( "SUCCEED showcase" ) {
    int I = 1;
    SUCCEED( "I is " << I );
}
```

---

### `STATIC_REQUIRE`

> [Catch 2.4.2 で導入](https://github.com/catchorg/Catch2/issues/1362)

`STATIC_REQUIRE(expr)` は `static_assert` と似ていますが、Catch2 に成功として記録されます。  
`CATCH_CONFIG_RUNTIME_STATIC_REQUIRE` を定義することで、実行時チェックに変更することも可能です。

#### 例：
```cpp
TEST_CASE("STATIC_REQUIRE showcase", "[traits]") {
    STATIC_REQUIRE( std::is_void<void>::value );
    STATIC_REQUIRE_FALSE( std::is_void<int>::value );
}
```

---

## テストケース関連のマクロ

### `METHOD_AS_TEST_CASE`

クラスのメンバ関数をテストケースとして登録するマクロです：

```cpp
METHOD_AS_TEST_CASE( クラス::メソッド, "説明", "[タグ]" )
```

#### 例：
```cpp
class TestClass {
    std::string s;

public:
    TestClass() : s("hello") {}

    void testCase() {
        REQUIRE( s == "hello" );
    }
};

METHOD_AS_TEST_CASE( TestClass::testCase, "クラスのメソッドをテストケースとして使用", "[class]" )
```

---

### `REGISTER_TEST_CASE`

通常の関数（`void()` 型）を Catch2 のテストケースとして登録します。

```cpp
REGISTER_TEST_CASE( 関数名, "説明 [タグ]" );
```

#### 例：
```cpp
void someFunction() {
    REQUIRE(true);
}

REGISTER_TEST_CASE( someFunction, "手動登録テスト", "[manual]" );
```

> 注意：この登録は Catch2 セッションの初期化前に実行されている必要があります。  
> 通常はグローバルコンストラクタか、ユーザー定義の `main()` の前に行います。

---

### `ANON_TEST_CASE`

名前の自動生成された匿名の `TEST_CASE` を定義します。  
名前を考える必要がなく簡便ですが、テストの特定が困難になる可能性があります。

#### 例：
```cpp
ANON_TEST_CASE() {
    SUCCEED("匿名テストケースからの成功");
}
```

---

### `DYNAMIC_SECTION`

> Catch 2.3.0 で導入

`DYNAMIC_SECTION` は `SECTION` の動的バージョンです。  
`<<` 演算子を使ってセクション名を構築でき、ループやジェネレータと組み合わせて柔軟な名前付けが可能です。

#### 例：
```cpp
TEST_CASE( "ループによるセクション名の生成" ) {
    int a = 1;

    for( int b = 0; b < 10; ++b ) {
        DYNAMIC_SECTION( "b の値: " << b ) {
            CHECK( b > a );
        }
    }
}
```

---

[ホームへ戻る](Readme.md)
