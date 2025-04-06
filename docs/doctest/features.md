## 特徴と設計目標

**doctest** は最初から、**軽量**で**侵入的でない（非侵入的）**ことを目指して設計されています。以下の特徴はその理念を体現しています。

---

## 非侵入的（透明）:

- [**`DOCTEST_CONFIG_DISABLE`**](configuration.md#doctest_config_disable) を定義することで、テスト関連のコードを実行ファイルから完全に除去可能
- 単一ヘッダでとても小さく、統合が簡単
- コンパイル時間への影響が**非常に小さい** — ヘッダをインクルードするだけでのオーバーヘッドは [**約 25ms**](benchmarks.md#cost-of-including-the-header)
- [**最速クラス**のアサートマクロ](benchmarks.md#cost-of-an-assertion-macro) — 例：アサート5万個でも30秒未満（場合によっては10秒以下）でコンパイル
- ヘッダをインクルードしても余計な標準ヘッダなどを引き込まない（実装部でのみ必要なものを引き込む）
- すべてのシンボルは `doctest` 名前空間内に定義（内部実装は `doctest::detail` にある）
- すべてのマクロには接頭辞あり（未接頭辞版もオプションで利用可能） → [**設定参照**](configuration.md)
- どのコンパイラでも**警告ゼロ**（最も厳しい警告レベルでも）：
    - Clang: `-Weverything -pedantic`
    - GCC: `-Wall -Wextra -pedantic` + [**35以上**の追加警告](../../scripts/cmake/common.cmake#L84)
    - MSVC: `/Wall`
- コマンドラインで認識しないオプションがあってもエラーにならず、接頭辞で他ライブラリとの干渉を防止可能
- [`main()` を自前で用意する場合](main.md)、コマンドラインではなく**コード上でオプション設定が可能**
- doctest のインクルードによって、コンパイラの警告設定が変更されたり残ったりしない

---

## 高い移植性:

※一部は古い情報を含みます

- 標準に準拠した **C++11** コード（C++98/03 向けには [**1.2.9**](https://github.com/doctest/doctest/tree/1.2.9) を利用）
- 以下のコンパイラ群でテスト済み：
    - **GCC**: 4.8〜12
    - **Clang**: 3.5〜15（Xcode 10+ 含む）
    - **MSVC**: 2015, 2017, 2019, 2022（32bit モード含む）
- GitHub Actions による自動テスト：
    - 最も厳しい警告レベルで **警告をエラー化**しつつテスト
    - 静的解析（Cppcheck、Clang-Tidy、Coverity Scan、OCLint、Visual Studio Analyzer）
    - 出力結果が以前の正常実行とバイト単位で一致するかを検証
    - Debug/Release モードでのビルドと実行
    - Linux 上で valgrind による検証（※macOS 上では非対応）
    - アドレス/未定義動作/スレッドサニタイザによるテスト
    - UNIX/Windows 含め **300以上の構成**で継続的にテスト

---

## その他の特徴:

- 始めるのがとても簡単 — 単一ヘッダ → [**チュートリアルはこちら**](tutorial.md)
- ヘッダ1つ、ポータブルで軽量、非侵入的 → [**ベンチマーク**](benchmarks.md) 参照
- [**`DOCTEST_CONFIG_DISABLE`**](configuration.md#doctest_config_disable) により、テスト関連をすべて除去可能
- テスト登録は自動で行われる（手動でリストに追加不要）
- [**Subcase**](tutorial.md#test-cases-and-subcases)：テストケース内で共通処理を簡潔に記述可能（[**Fixture**](testcases.md#test-fixtures) もサポート）
- [**テンプレートテストケース**](parameterized-tests.md#templated-test-cases---parameterized-by-type)：型ごとの繰り返しテストに対応
- [**ログマクロ**](logging.md)：assert 失敗時に変数の値などを出力（遅延文字列化・可能な限り非アロケート）
- クラッシュハンドリング（UNIXでは signal、Windowsでは SEH）
- [**スレッドセーフ**](faq.md#is-doctest-thread-aware)：1テストケース内でのマルチスレッド使用が可能 → [**例**](../../examples/all_features/concurrency.cpp)
- 拡張可能な [**レポーターシステム**](reporters.md)（イベントリスナーにも利用可能）
- 全コンパイラでの出力が同一（バイト単位で一致）
- 実行ファイル／共有ライブラリ間でテストランナーを共有可能 → [**例**](../../examples/executable_dll_and_plugin/)
- [**BDD スタイルテスト**](testcases.md) をサポート
- C++演算子ベースの [**assert マクロ**](assertions.md)：式を分解して左右の値もログ出力
- [**テスト以外の文脈でも assert が使用可能**](assertions.md#using-asserts-out-of-a-testing-context) → [**例**](../../examples/all_features/asserts_used_outside_of_tests.cpp)
- [**例外に関するアサート**](assertions.md#exceptions) に対応
- 浮動小数点の比較 → [**`Approx()`**](assertions.md#floating-point-comparisons) ヘルパーあり
- ユーザー定義型の [**文字列化**](stringification.md) に柔軟対応（[**例外型も対応**](stringification.md#translating-exceptions)）
- テストケースの [**テストスイート分類**](testcases.md#test-suites) が可能
- [**デコレータ**](testcases.md#decorators)：`description`、`skip`、`may_fail`、`should_fail`、`expected_failures`、`timeout` など
- 例外と RTTI を使用しないビルドにも対応 → [**`DOCTEST_CONFIG_NO_EXCEPTIONS`**](configuration.md#doctest_config_no_exceptions)
- 高機能な [**コマンドラインオプション**](commandline.md)
- テストごとの実行時間を計測・表示可能
- テスト／ファイル名／スイート名による [**フィルタリング実行**](commandline.md) が可能（ワイルドカード対応）
- サブケースもネストレベル単位で [**フィルタリング実行**](commandline.md) 可能
- テスト失敗時に Windows/macOS のデバッガに自動ブレーク可能
- Visual Studio の出力ウィンドウとの統合に対応
- [**`DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN`**](main.md#doctest_config_implement_with_main) により自動で `main()` を生成可能
- ヘッダにテストを書いても、登録は1回だけ（重複なし）
- テストの [**範囲実行（range-based execution）**](commandline.md) に対応 → [**Python スクリプト例**](../../examples/range_based_execution.py)
- 追加機能は [**拡張ヘッダ**](extensions.md) として分離されており、基本ヘッダに影響なし
- カラー出力サポート（対応ターミナルで自動判定）
- テスト実行順の制御が可能
- 異なる `doctest::Context` を何度も実行可能
- 現在テスト中かどうかの判定 → `doctest::is_running_in_test`
- [**doctest_discover_tests(<target>)**](../../scripts/cmake/doctest.cmake) により CTest への登録が可能

---

今後の重要な機能追加の予定は [**roadmap**](https://github.com/doctest/doctest/issues/600) を参照してください。

---

[ホームへ戻る](readme.md#reference)
