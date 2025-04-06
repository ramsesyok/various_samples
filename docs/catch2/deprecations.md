# 非推奨および今後の変更
## 非推奨（Deprecations）

### `--list-*` の戻り値

`--list-*` 系のコマンドライン引数は、これまで検出されたテストやタグの数を終了コードとして返していましたが、今後は「成功で 0、失敗で非 0」という通常の方式に変更されます。

---

### `--list-test-names-only`

`--list-test-names-only` コマンドライン引数は削除される予定です。

---

### `ANON_TEST_CASE`

`ANON_TEST_CASE` は非推奨となり、削除される予定です。  
これは、引数なしの `TEST_CASE` で完全に代替可能なためです。

---

### タグ中のセカンダリ記述（説明文）

現在、`TEST_CASE`（など）のタグ部分に、タグ以外のテキストを記述することができ、これはテストの「説明文」として扱われます。  
ただし、この説明は `--list-tests -v high` を使用した場合にのみ表示され、その他の用途では使用されていません。

この機能は正式に文書化されておらず、内部処理も複雑になるため、削除される予定です。

---

### `SourceLineInfo::empty()`

空の `SourceLineInfo` を必要とする理由がないため、このメソッドは削除されます。

---

### 既に合成されたマッチャの左辺値をさらに合成すること

この機能には 2 年以上前から重大なバグがあるにもかかわらず、バグ報告がなかったことと、実装の簡素化のため、将来的にはこのコードはコンパイルできなくなります：

```cpp
auto m1 = Contains("string");
auto m2 = Contains("random");
auto composed1 = m1 || m2;
auto m3 = Contains("different");
auto composed2 = composed1 || m3; // ← これが NG になる
REQUIRE_THAT(foo(), !composed1);
REQUIRE_THAT(foo(), composed2);
```

今後は以下のように書く必要があります：

```cpp
auto m1 = Contains("string");
auto m2 = Contains("random");
auto m3 = Contains("different");
REQUIRE_THAT(foo(), !(m1 || m2));
REQUIRE_THAT(foo(), m1 || m2 || m3);
```

---

### `ParseAndAddCatchTests.cmake`

CMake/CTest 用の `ParseAndAddCatchTests.cmake` は非推奨です。  
これは C++ コードを正規表現で解析していたため精度に限界がありましたが、代替手段として `Catch.cmake` の `catch_discover_tests` を使えば、CMake ターゲットからコマンドラインで直接テスト情報を取得できます。

---

## 今後の変更予定（Planned changes）

### レポーターの詳細表示オプション（verbosity）

現在の実装では、レポーターが要求された詳細レベルに対応しているかどうかを事前にチェックしていますが、これは根本的に誤った設計であると判断され、変更されます。  
将来的には、レポーター自身が `verbosity` を解釈し、未対応の場合もエラーではなく「警告」にとどめる方針になります。

---

### `--list-*` の出力形式

`--list-*` オプションの出力は、すべてレポーター経由で処理されるようになります。  
例えば、XML レポーターを指定すれば XML 形式、Console レポーターならこれまで通りの人間向けテキスト形式になります。

---

### `CHECKED_IF` および `CHECKED_ELSE`

これらのマクロは、失敗してもテスト自体を失敗としないように変更されます。  
具体的には `Catch::ResultDisposition::SuppressFail` フラグが追加され、`else` 節がより実用的になります。

---

### `[.]` タグと除外指定の意味変更

現在、以下のようなテストがあると：

```cpp
TEST_CASE("A", "[.][foo]") {}
TEST_CASE("B", "[.][bar]") {}
```

- `[foo]` を指定 → テスト "A" が実行される
- `~[foo]` を指定 → "B" も実行される（隠しテストなのに！）
- `~[baz]` を指定 → 両方実行される

この挙動は混乱を招きやすいため、**今後は明示的にマッチしない限り隠しテストは実行されない**ように変更されます。

---

### コンソールカラー API の変更

Catch2 のカラー出力 API が変更され、適用先となる出力ストリームを引数として渡すようになります。

---

### `PredicateMatcher` の型消去の廃止

現在 `PredicateMatcher` は型消去のために `std::function` を使用していますが、これはコンパイル時間の負荷が大きいため、今後は使用されなくなります。  
その代わり、述語関数の型情報は `PredicateMatcher` のテンプレート型として残ることになります。

---

[ホームに戻る](Readme.md)