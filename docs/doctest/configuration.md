# 設定（Configuration）

**doctest** は「そのまま動く」ことを目指して設計されていますが、さまざまな構成オプションによってビルド方法を柔軟に変更できます。

これらの識別子（マクロ定義）は、`#include "doctest.h"` の**前に定義**する必要があります。

`グローバルに定義` とは、バイナリ（実行ファイル / DLL）のすべてのソースファイルに対して適用することを意味します。

---

## よく使われる構成オプション

### `DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN`

```cpp
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include "doctest.h"
```

- このファイルに `main()` 関数が自動生成されます。
- テスト実行だけのバイナリを作るときに便利です。
- **1つのソースファイルのみに定義してください。**

---

### `DOCTEST_CONFIG_IMPLEMENT`

- `main()` 関数を自分で定義する場合に使用。
- 設定変更やアプリケーション統合時に有用。
- **1つのソースファイルのみに定義してください。**

---

### `DOCTEST_CONFIG_DISABLE`

- **テスト関連コードをすべて除去**します（コンパイルもされずバイナリにも含まれません）。
- リリースビルド時に便利。
- **グローバルに定義してください。**

---

## DLL関連

### `DOCTEST_CONFIG_IMPLEMENTATION_IN_DLL`

- テストランナーを DLL に分離したい場合に使用。
- 複数バイナリで共通のテストランナーを使いたいときに便利。

---

## マクロ名関連

### `DOCTEST_CONFIG_NO_SHORT_MACRO_NAMES`

- `CHECK` などの短いマクロ名を無効にして、`DOCTEST_CHECK` のような完全名のみを使えるようにします。

---

## 比較や文字列化に関するオプション

- `DOCTEST_CONFIG_TREAT_CHAR_STAR_AS_STRING`  
  → `char*` を文字列として扱う（`strcmp` で比較される）。

- `DOCTEST_CONFIG_REQUIRE_STRINGIFICATION_FOR_ALL_USED_TYPES`  
  → すべての型に明示的な stringification を強制。

- `DOCTEST_CONFIG_DOUBLE_STRINGIFY`  
  → カスタム `toString()` の出力をもう一度文字列化したいときに使用。

---

## アサート最適化・高速化関連

- `DOCTEST_CONFIG_SUPER_FAST_ASSERTS`  
  → アサートのコンパイル速度を大幅に向上。`DOCTEST_CONFIG_NO_TRY_CATCH_IN_ASSERTS` も含まれる。

- `DOCTEST_CONFIG_ASSERTION_PARAMETERS_BY_VALUE`  
  → アサートの引数を const 参照ではなく値渡しにする。

- `DOCTEST_CONFIG_ASSERTS_RETURN_VALUES`  
  → アサート結果を `bool` として返す。if文などに組み込み可能。

- `DOCTEST_CONFIG_EVALUATE_ASSERTS_EVEN_WHEN_DISABLED`  
  → `DISABLE` 状態でもアサート条件を評価（副作用維持）。

---

## 例外関連

- `DOCTEST_CONFIG_NO_TRY_CATCH_IN_ASSERTS`  
  → アサート内の `try/catch` を無効化。

- `DOCTEST_CONFIG_NO_EXCEPTIONS`  
  → フレームワーク全体から例外処理を除去。`REQUIRE` や `THROWS` 系マクロが削除されます。

- `DOCTEST_CONFIG_NO_EXCEPTIONS_BUT_WITH_ALL_ASSERTS`  
  → 上記と併用して `REQUIRE` などを `CHECK` のように機能させたい場合に使用。

---

## コンソール出力の制御

- `DOCTEST_CONFIG_COLORS_NONE`  
  → 色付き出力を完全に無効化。

- `DOCTEST_CONFIG_COLORS_WINDOWS` / `DOCTEST_CONFIG_COLORS_ANSI`  
  → 出力方式を明示的に指定。

---

## OS/環境依存の例外処理

- `DOCTEST_CONFIG_WINDOWS_SEH`  
  → Windows の SEH (構造化例外ハンドリング) を有効化（MSVC 向け）。

- `DOCTEST_CONFIG_NO_WINDOWS_SEH`  
  → 上記を無効化。

- `DOCTEST_CONFIG_POSIX_SIGNALS`  
  → POSIX シグナルによるクラッシュ検出を有効化。

- `DOCTEST_CONFIG_NO_POSIX_SIGNALS`  
  → 上記を無効化。

---

## コマンドライン関連

- `DOCTEST_CONFIG_OPTIONS_PREFIX`  
  → `--dt-` 以外のプレフィックスに変更できます（例：`--selftest-`）。

- `DOCTEST_CONFIG_NO_UNPREFIXED_OPTIONS`  
  → プレフィックスなしオプション（`--help` など）を無効化。

---

## その他のオプション

- `DOCTEST_CONFIG_NO_COMPARISON_WARNING_SUPPRESSION`  
  → `signed/unsigned` 比較の警告抑制を無効にする。

- `DOCTEST_CONFIG_NO_CONTRADICTING_INLINE`  
  → `inline` + `noinline` の併用を避ける（特定のコンパイラ対策）。

- `DOCTEST_CONFIG_NO_INCLUDE_IOSTREAM`  
  → `<iostream>` を含めない。標準出力せずに独自の出力を使いたい場合。

- `DOCTEST_CONFIG_HANDLE_EXCEPTION`  
  → 例外の出力方法をカスタマイズしたい場合に定義。

- `DOCTEST_CONFIG_INCLUDE_TYPE_TRAITS`  
  → `<type_traits>` を明示的に含める（`Approx` 用）。

- `DOCTEST_CONFIG_NO_MULTITHREADING`  
  → マルチスレッド処理を完全に無効化。

- `DOCTEST_CONFIG_NO_MULTI_LANE_ATOMICS`  
  → アトミック操作を単一レーンに制限（単一スレッド高速化）。

---

[ホームに戻る](readme.md#reference)
