# コンパイル時設定

Catch は「できるだけ何も設定しなくても動く」ように設計されています。  
大抵の場合、以下のように `main()` を自動生成させるだけで十分です：

```cpp
#define CATCH_CONFIG_MAIN
```

ただし、細かい制御が必要な場合は、Catch が用意しているマクロ群を使って構成を変更できます。

---

## main()/ 実装ファイルの指定

```cpp
CATCH_CONFIG_MAIN      // main() を定義し、Catch の実装を含める
CATCH_CONFIG_RUNNER    // Catch の実装だけを含める（main は自分で書く）
```

Catch はヘッダオンリーですが、内部的にはインターフェースと実装を分けています。  
**プロジェクト内で1つだけ、これらのマクロを使って Catch の実装を含める必要があります。**

---

## レポーター / リスナーのインターフェース

```cpp
CATCH_CONFIG_EXTERNAL_INTERFACES
```

カスタムレポーターやリスナーを実装するために必要な定義を取り込むマクロです。  
`CATCH_CONFIG_MAIN` または `CATCH_CONFIG_RUNNER` を定義したときにも自動的に含まれます。

---

## Catch マクロの接頭辞

```cpp
CATCH_CONFIG_PREFIX_ALL
```

`TEST_CASE`, `REQUIRE` などのマクロは簡潔ですが、他のライブラリやシステムと衝突することがあります。  
このマクロを定義すると、すべてのマクロに `CATCH_` が付きます（例：`CATCH_TEST_CASE`）。

---

## ターミナルのカラー表示

```cpp
CATCH_CONFIG_COLOUR_NONE      // カラー無効
CATCH_CONFIG_COLOUR_WINDOWS   // Win32 API のカラーを強制
CATCH_CONFIG_COLOUR_ANSI      // ANSI エスケープシーケンスを強制
```

カラー出力の方式を制御します。デフォルトでは自動検出されます。  
`isatty()` を利用するために `"unistd.h"` が必要なことに注意してください。

---

## コンソール幅

```cpp
CATCH_CONFIG_CONSOLE_WIDTH = x
```

Catch の出力幅を設定できます（デフォルトは 80）。  
折り返しやインデントの整合性を保つために必要です。

---

## 標準出力

```cpp
CATCH_CONFIG_NOSTDOUT
```

標準出力 (`std::cout` など) を使用せず、自前の関数を定義できます：

```cpp
std::ostream& Catch::cout();
std::ostream& Catch::cerr();
std::ostream& Catch::clog();
```

[使用例はこちら](../examples/231-Cfg-OutputStreams.cpp)

---

## フォールバックの文字列化関数

```cpp
CATCH_CONFIG_FALLBACK_STRINGIFIER
```

`StringMaker` や `operator<<` が定義されていない型の文字列化処理を自前の関数で行う場合に使用します。  
定義された関数は `std::string func(any_type)` のように任意の型に対応できる必要があります。

---

## デフォルトレポーター

```cpp
CATCH_CONFIG_DEFAULT_REPORTER
```

デフォルトで使用するレポーターの名前（文字列）を指定します。  
例えば `"junit"` や `"compact"` など。

---

## C++11 に関する切り替え

```cpp
CATCH_CONFIG_CPP11_TO_STRING
```

`std::to_string` を使うかどうかの制御。  
Android ではデフォルトで `stringstream` を使用しますが、他の環境では `to_string` を使います。

---

## C++17 に関する切り替え

```cpp
CATCH_CONFIG_CPP17_UNCAUGHT_EXCEPTIONS
CATCH_CONFIG_CPP17_STRING_VIEW
CATCH_CONFIG_CPP17_VARIANT
CATCH_CONFIG_CPP17_OPTIONAL
CATCH_CONFIG_CPP17_BYTE
```

これらのマクロで C++17 機能の有効/無効を明示的に指定できます。  
自動検出がうまくいかない環境では便利です。

---

## その他の切り替え設定 { #other-toggles }

```cpp
CATCH_CONFIG_COUNTER                    // __COUNTER__ を使用して一意な名前を生成
CATCH_CONFIG_WINDOWS_SEH               // Windows の SEH（構造化例外）対応
CATCH_CONFIG_FAST_COMPILE              // コンパイル速度重視モード
CATCH_CONFIG_DISABLE_MATCHERS          // Matcher を無効化
CATCH_CONFIG_POSIX_SIGNALS             // POSIX シグナルハンドリング
CATCH_CONFIG_WINDOWS_CRTDBG            // Windows のヒープリーク検出機能を有効化
CATCH_CONFIG_DISABLE_STRINGIFICATION   // 式の文字列化を無効化
CATCH_CONFIG_DISABLE                   // アサーションもテスト登録も完全無効
CATCH_CONFIG_WCHAR                     // wchar_t 対応を有効化
CATCH_CONFIG_EXPERIMENTAL_REDIRECT     // stdout/stderr の新しい捕捉方法を有効化
CATCH_CONFIG_ENABLE_BENCHMARKING       // ベンチマーク機能を有効化
CATCH_CONFIG_USE_ASYNC                 // ベンチマーク処理の並列化を有効化
CATCH_CONFIG_ANDROID_LOGWRITE          // Android のログシステムを使用
CATCH_CONFIG_GLOBAL_NEXTAFTER          // `std::nextafter` の代わりにグローバル関数を使用
```

⚠️ `CATCH_CONFIG_DISABLE` を使うと、アサーションもテスト登録もすべて無効になります。

---

## Windows ヘッダの無駄な定義の抑制

```cpp
CATCH_CONFIG_NO_NOMINMAX
CATCH_CONFIG_NO_WIN32_LEAN_AND_MEAN
```

`windows.h` によるグローバル汚染を防ぐため、`NOMINMAX` や `WIN32_LEAN_AND_MEAN` を定義しますが、これを無効にするマクロです。

---

## 標準ライブラリ型の文字列化を有効化

```cpp
CATCH_CONFIG_ENABLE_PAIR_STRINGMAKER
CATCH_CONFIG_ENABLE_TUPLE_STRINGMAKER
CATCH_CONFIG_ENABLE_CHRONO_STRINGMAKER
CATCH_CONFIG_ENABLE_VARIANT_STRINGMAKER
CATCH_CONFIG_ENABLE_OPTIONAL_STRINGMAKER
CATCH_CONFIG_ENABLE_ALL_STRINGMAKERS
```

`std::pair`, `std::tuple`, `std::chrono`, `std::variant`, `std::optional` などの文字列化を有効化します。

---

## 例外機能の無効化

```cpp
CATCH_CONFIG_DISABLE_EXCEPTIONS
```

例外の代わりに `std::terminate()` を使用するように切り替える実験的な機能です。  
また、代替動作を定義したい場合は以下を定義します：

```cpp
CATCH_CONFIG_DISABLE_EXCEPTIONS_CUSTOM_HANDLER

namespace Catch {
    [[noreturn]]
    void throw_exception(std::exception const&);
}
```

---

## Catch のデバッグブレークの上書き (`-b`)

```cpp
CATCH_BREAK_INTO_DEBUGGER()
```

Catch2 のデバッグ中断機能をカスタムコードに置き換えたい場合に使用します。

---

[ホームに戻る](Readme.md)
