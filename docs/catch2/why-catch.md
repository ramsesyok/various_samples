以下は、「Why do we need yet another C++ test framework?（なぜまた新しいC++テストフレームワークが必要なのか）」の日本語訳です。見出しは日本語にし、アンカー（id）は英語のまま保持しています。

---

<a id="top"></a>  
# なぜまた新しい C++ テストフレームワークが必要なのか？

良い質問です。C++ にはすでに多くの確立されたフレームワークがあります（以下は一例です）：

- [Google Test](http://code.google.com/p/googletest/)
- [Boost.Test](http://www.boost.org/doc/libs/1_49_0/libs/test/doc/html/index.html)
- [CppUnit](http://sourceforge.net/apps/mediawiki/cppunit/index.php?title=Main_Page)
- [Cute](http://www.cute-test.com)
- [その他多数](http://en.wikipedia.org/wiki/List_of_unit_testing_frameworks#C.2B.2B)

では、Catch はこれらと何が違うのでしょうか？キャッチーな名前以外に。

## <a id="key-features"></a>主な特徴

- とにかく **素早く簡単に始められる**：`catch.hpp` をダウンロードして `#include` すればすぐに使えます。
- **外部依存なし**：C++11 が使えて、標準ライブラリがあれば OK。
- テストケースは **自己登録型の関数**（またはメソッド）として記述します。
- テストケースを **セクション（Section）に分割**し、各セクションは独立して実行されます（フィクスチャ不要）。
- BDDスタイルの **Given-When-Then** 構文にも対応。従来のユニットテスト記法と併用可能。
- 比較用のアサーションマクロは基本的に1つだけ。比較には **標準のC/C++演算子**を使い、式全体が分解されて出力されます。
- テスト名は **自由な文字列**で指定できます（識別子にする必要なし）。

## <a id="other-core-features"></a>その他の基本機能

- テストにはタグを付けられ、任意のグループだけを簡単に実行可能。
- Windows や Mac では **失敗時にデバッガでブレーク**するオプションあり。
- 出力は **モジュール型のレポーターオブジェクト**を使用。テキスト・XML 出力対応。独自レポーターの追加も簡単。
- **JUnit 形式の XML 出力**に対応しており、CI サーバー等のツールと連携可能。
- **main 関数はデフォルト提供**。自作の main() を書くことで GUI テストランナー等への統合も可能。
- **コマンドラインパーサー付き**。main() を自作しても使えます。
- **Catch 自体も Catch でテスト可能**。
- アサーションの代替マクロにより、**失敗してもテストを中断しない**スタイルも選べます。
- **浮動小数点の誤差を許容する比較**が組み込み済み（`Approx()` 構文）。
- 内部用／公開用のマクロを分離し、**名前の衝突を最小限に抑制**。
- **マッチャー（Matchers）**により、柔軟な比較が可能。

## <a id="who-uses-catch"></a>Catch を使っているのは誰？

[Catch を使っているオープンソースプロジェクトの一覧はこちら](opensource-users.md#top)。

Catch の使い方をざっと確認したい場合は [チュートリアル](tutorial.md#top) をどうぞ。

---

[ホームへ戻る](Readme.md#top)