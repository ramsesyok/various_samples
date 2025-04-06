# パラメータ化テストケース（Parameterized test cases）

doctest では、テストケースを型（Type）または間接的に値（Value）によってパラメータ化することができます。

---

## 値によるパラメータ化（Value-parameterized test cases）

将来的には正式な対応が予定されていますが、現在のところ、値を使ったデータ駆動テストには次の2つの方法があります。

### 1. ヘルパー関数を使う方法

```cpp
void doChecks(int data) {
    // アサーションを実行
}

TEST_CASE("テスト名") {
    std::vector<int> data = {1, 2, 3, 4, 5, 6};

    for (auto& i : data) {
        CAPTURE(i); // 現在のデータをログに出力
        doChecks(i);
    }
}
```

**デメリット：**

- 途中で例外が投げられたり `REQUIRE` に失敗するとテストケース全体が終了して、後続データのチェックができません。
- `CAPTURE()` や `INFO()` を手動で使ってログを出力する必要があります。
- データ生成も手動で行う必要があり、ボイラープレートが多くなります。

---

### 2. `SUBCASE()` を使う方法

```cpp
TEST_CASE("テスト名") {
    int data;
    SUBCASE("") { data = 1; }
    SUBCASE("") { data = 2; }

    CAPTURE(data);
    // アサーションを実行
}
```

**デメリット：**

- 多くの入力値に対してはスケーラビリティが低く、手動記述が非現実的です。
- こちらもログ出力は `CAPTURE()` などで手動。

---

### 3. マクロで簡略化する方法

```cpp
#define DOCTEST_VALUE_PARAMETERIZED_DATA(data, data_container)                                  \
    static size_t _doctest_subcase_idx = 0;                                                     \
    std::for_each(data_container.begin(), data_container.end(), [&](const auto& in) {          \
        DOCTEST_SUBCASE((std::string(#data_container "[") +                                     \
                        std::to_string(_doctest_subcase_idx++) + "]").c_str()) { data = in; }  \
    });                                                                                         \
    _doctest_subcase_idx = 0;
```

使用例：

```cpp
TEST_CASE("テスト名") {
    int data;
    std::list<int> data_container = {1, 2, 3, 4};

    DOCTEST_VALUE_PARAMETERIZED_DATA(data, data_container);

    printf("%d\n", data);
}
```

出力例：

```
1
2
3
4
```

**制限事項：**

- 同じ `{}` ブロック内で他の `SUBCASE()` と混在させると動作が不安定になります。
- `SUBCASE()` 内で使うのが安全です。

---

## 型によるパラメータ化（Templated test cases）

同じロジックを複数の型に対して繰り返しテストしたい場合、テンプレートを使ったパラメータ化が有効です。

### 方法1：型リストを直接渡す

```cpp
TEST_CASE_TEMPLATE("符号付き整数の動作", T, char, short, int, long long int) {
    T var = T();
    --var;
    CHECK(var == -1);
}
```

### 方法2：定義と呼び出しを分ける

```cpp
TEST_CASE_TEMPLATE_DEFINE("符号付き整数の動作", T, test_id) {
    T var = T();
    --var;
    CHECK(var == -1);
}

TEST_CASE_TEMPLATE_INVOKE(test_id, char, short, int, long long int);
TEST_CASE_TEMPLATE_APPLY(test_id, std::tuple<float, double>);
```

このようにすれば、同じテストロジックを複数の型に簡単に適用できます。

```text
例： "符号付き整数の動作<int>" のように出力されます。
```

---

### 型の文字列化（TYPE_TO_STRING）

標準のプリミティブ型（`int`, `bool`, `float` など）以外の型を使用する場合は、型を文字列化するために `TYPE_TO_STRING()` マクロを使用します。

```cpp
TYPE_TO_STRING(std::vector<int>);
```

このマクロは現在のソースファイル内でのみ有効なため、複数ファイルで使用する場合はヘッダに書く必要があります。

> doctest は極力高速なコンパイルを実現するため、`<typeinfo>` などを含まない設計になっています。

---

### その他の注意点：

- 同じ型に対して複数回インスタンス化することは制限されていません（自分で防ぐ必要があります）。
- 型の文字列化がなくてもテストは実行されます（名前は `<...>` になります）。
- 複数の型をまとめて 1つにするには、以下のような `TypePair` 構造体を使用します：

```cpp
template <typename first, typename second>
struct TypePair {
    using A = first;
    using B = second;
};

#define pairs \
    TypePair<int, char>, \
    TypePair<char, int>

TEST_CASE_TEMPLATE("複数型のテスト", T, pairs) {
    using T1 = typename T::A;
    using T2 = typename T::B;
    // T1 と T2 を使用してテスト
}
```

---

- 詳しくは [**テンプレートテストの例**](../../examples/all_features/templated_test_cases.cpp) をご覧ください。

---

[ホームに戻る](readme.md#reference)
