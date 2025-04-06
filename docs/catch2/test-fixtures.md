# テストフィクスチャ（Test fixtures）

## テストフィクスチャの定義

Catch では、テストを1つのテストケースの中にセクションとしてまとめることが可能ですが、従来の「テストフィクスチャ」スタイルでテストを整理した方が便利な場合もあります。Catch はこのスタイルにも完全対応しています。

テストフィクスチャは、シンプルなクラスまたは構造体として定義します：

```cpp
class UniqueTestsFixture {
  private:
    static int uniqueID;
  protected:
    DBConnection conn;
  public:
    UniqueTestsFixture() : conn(DBConnection::createConnection("myDB")) {
    }
  protected:
    int getID() {
      return ++uniqueID;
    }
};

int UniqueTestsFixture::uniqueID = 0;

TEST_CASE_METHOD(UniqueTestsFixture, "Create Employee/No Name", "[create]") {
  REQUIRE_THROWS(conn.executeSQL("INSERT INTO employee (id, name) VALUES (?, ?)", getID(), ""));
}
TEST_CASE_METHOD(UniqueTestsFixture, "Create Employee/Normal", "[create]") {
  REQUIRE(conn.executeSQL("INSERT INTO employee (id, name) VALUES (?, ?)", getID(), "Joe Bloggs"));
}
```

この2つのテストケースは `UniqueTestsFixture` を継承した独自のクラスとして生成されます。  
そのため、`getID()` メソッドや `conn` メンバーにアクセスできます。

- 両方のテストケースが同じ方法で `DBConnection` を作成できます（DRY 原則）。
- 作成される ID は一意であり、テストの実行順序に依存しません。

---

Catch2 では、`TEMPLATE_TEST_CASE_METHOD` や `TEMPLATE_PRODUCT_TEST_CASE_METHOD` も用意されています。  
これらを使うことで、テンプレート化されたフィクスチャを複数の型でテストできます。

注意点：

- `TEST_CASE_METHOD` と異なり、`TEMPLATE_TEST_CASE_METHOD` と `TEMPLATE_PRODUCT_TEST_CASE_METHOD` では **タグの指定が必須** です。
- テンプレート型が複数のテンプレート引数を持つ場合は、必ず `()` で囲む必要があります。例：`(std::map<int, std::string>)`
- `TEMPLATE_PRODUCT_TEST_CASE_METHOD` で型リストの各項目が複数型から成る場合、さらに外側の括弧で囲む必要があります。例：`(std::map, std::pair)`, `((int, float), (char, double))`

### 例：

```cpp
template< typename T >
struct Template_Fixture {
    Template_Fixture(): m_a(1) {}

    T m_a;
};

TEMPLATE_TEST_CASE_METHOD(Template_Fixture,"TEMPLATE_TEST_CASE_METHOD によるテスト", "[class][template]", int, float, double) {
    REQUIRE( Template_Fixture<TestType>::m_a == 1 );
}

template<typename T>
struct Template_Template_Fixture {
    Template_Template_Fixture() {}

    T m_a;
};

template<typename T>
struct Foo_class {
    size_t size() {
        return 0;
    }
};

TEMPLATE_PRODUCT_TEST_CASE_METHOD(Template_Template_Fixture, "TEMPLATE_PRODUCT_TEST_CASE_METHOD によるテスト", "[class][template]", (Foo_class, std::vector), int) {
    REQUIRE( Template_Template_Fixture<TestType>::m_a.size() == 0 );
}
```

> `TEMPLATE_TEST_CASE_METHOD` や `TEMPLATE_PRODUCT_TEST_CASE_METHOD` に指定できる型数には上限がありますが、非常に高いため、通常の使用で制限に達することはありません。

---

## シグネチャベースのパラメータ付きテストフィクスチャ

> [Catch 2.8.0 で導入](https://github.com/catchorg/Catch2/issues/1609)

Catch2 では、非型テンプレートパラメータ（NTTP）を使用したフィクスチャに対応するため、  
`TEMPLATE_TEST_CASE_METHOD_SIG` および `TEMPLATE_PRODUCT_TEST_CASE_METHOD_SIG` が提供されています。

これらは `TEMPLATE_TEST_CASE_METHOD` および `TEMPLATE_PRODUCT_TEST_CASE_METHOD` に似ていますが、  
追加で [シグネチャ](test-cases-and-sections.md#signature-based-parametrised-test-cases) を指定します。

### 例：

```cpp
template <int V>
struct Nttp_Fixture{
    int value = V;
};

TEMPLATE_TEST_CASE_METHOD_SIG(Nttp_Fixture, "NTTP を使ったテスト", "[class][template][nttp]", ((int V), V), 1, 3, 6) {
    REQUIRE(Nttp_Fixture<V>::value > 0);
}

template<typename T>
struct Template_Fixture_2 {
    Template_Fixture_2() {}

    T m_a;
};

template< typename T, size_t V>
struct Template_Foo_2 {
    size_t size() { return V; }
};

TEMPLATE_PRODUCT_TEST_CASE_METHOD_SIG(Template_Fixture_2, "NTTP を含む PRODUCT テスト", "[class][template][product][nttp]", ((typename T, size_t S), T, S), (std::array, Template_Foo_2), ((int,2), (float,6)))
{
    REQUIRE(Template_Fixture_2<TestType>{}.m_a.size() >= 2);
}
```

---

## テンプレート型リストを使ったフィクスチャ

Catch2 では、型リストを使ったテンプレートフィクスチャ用に  
`TEMPLATE_LIST_TEST_CASE_METHOD` が用意されています。

このマクロは `TEMPLATE_TEST_CASE_METHOD` と同様に動作しますが、  
型リストのソースが異なります。`std::tuple`, `boost::mpl::list`, `boost::mp11::mp_list` などを使えます。

### 例：

```cpp
using MyTypes = std::tuple<int, char, double>;
TEMPLATE_LIST_TEST_CASE_METHOD(Template_Fixture, "std::tuple を使ったテスト", "[class][template][list]", MyTypes)
{
    REQUIRE( Template_Fixture<TestType>::m_a == 1 );
}
```

---

[ホームへ戻る](Readme.md)
