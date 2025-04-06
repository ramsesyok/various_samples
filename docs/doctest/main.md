## `main()` エントリーポイント

C++ における従来のテストの書き方は、テスト対象のコードとは別のソースファイルにテストコードを記述し、テスト専用の実行ファイルを作成するというスタイルでした。このようなシナリオでは、**doctest** が提供するデフォルトの `main()` 関数を利用するのが一般的です。

```cpp
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include "doctest.h"
```

これは、**1つのソースファイル内でのみ記述する必要があります**。また、他のコードを含まない専用ファイルとして用意するのが理想的です。

---

ただし、テスト実行時のオプションをコードから設定したい場合や、フレームワークを本番コードに統合したい場合など、より柔軟な制御が必要な場合には、デフォルトの `main()` 関数では不十分です。そのようなときは、[`DOCTEST_CONFIG_IMPLEMENT`](configuration.md#doctest_config_implement) を使用してください。

すべての [**コマンドラインオプション**](commandline.md) は以下のようにコードから設定可能です（ただし、フラグは論理的に意味をなさないため除きます）。フィルターは `doctest::Context` オブジェクトの `addFilter()` や `clearFilters()` メソッドでのみ追加・クリアできます。**特定のフィルターをコードから削除することはできません**。

```cpp
#define DOCTEST_CONFIG_IMPLEMENT
#include "doctest.h"

int main(int argc, char** argv) {
    doctest::Context context;

    // !!! これはデフォルト値やオーバーライド設定の方法を示す例です !!!

    // デフォルト設定
    context.addFilter("test-case-exclude", "*math*"); // 名前に "math" を含むテストケースを除外
    context.setOption("abort-after", 5);              // 5回の失敗後にテストを中断
    context.setOption("order-by", "name");            // テストケースを名前順に並び替え

    context.applyCommandLine(argc, argv);             // コマンドライン引数を適用

    // オーバーライド設定
    context.setOption("no-breaks", true);             // アサーション失敗時にデバッガでブレークしない

    int res = context.run(); // テスト実行

    if(context.shouldExit()) // フラグ（--exit など）の処理にはこのチェックが重要
        return res;          // テスト結果をそのまま返す

    int client_stuff_return_code = 0;
    // 本番コードと統合している場合は、ここにアプリケーション本体の処理を記述

    return res + client_stuff_return_code; // doctest の実行結果も最終的に返却
}
```

上記コード中の `context.shouldExit()` の呼び出しに注目してください。これは非常に重要で、クエリフラグが指定された場合（たとえば `--no-run` が `true` の場合など）に正しくアプリケーションを終了するために必要です。**このチェックはユーザーの責任で行う必要があります。**

---

### 共有ライブラリ（DLL）との連携

このフレームワークは、個別のバイナリ（実行ファイルや共有ライブラリ）ごとに独立したテストランナーを持たせて使用することも可能です。これにより、異なるバージョンの doctest を同時に使用することもできます。ただし、**すべてのバイナリからテストをまとめて実行し、結果を集約・要約する簡単な方法は存在しません**。

代替案としては、テストランナー（実装）を1つのバイナリ内にビルドし、それを他のコンポーネントと共有する方法もあります（つまり、テストレジストリを1つに統一する）。その場合、ユーザーがテストを書くのに必要な公開シンボル（doctest の forward 宣言）をエクスポートする必要があります。

詳細については、[`DOCTEST_CONFIG_IMPLEMENTATION_IN_DLL`](configuration.md#doctest_config_implementation_in_dll) の設定識別子や、[このサンプル](../../examples/executable_dll_and_plugin/)を参照してください。

---

[ホームに戻る](readme.md#reference)
