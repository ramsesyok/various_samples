# コマンドラインオプション（Command line）

**doctest** は、特にオプションを指定しなくても十分に機能しますが、より細かく制御したい場合のために多数のコマンドラインオプションが用意されています。

---

## オプションの種類

- **クエリフラグ（Query flags）**  
  テストケースの実行前に情報を表示して終了します。ユーザーが [`main()` を自作する場合](main.md) は、`doctest::Context` の `run()` 呼び出し後に `shouldExit()` を確認し、終了処理をする必要があります。

- **整数／文字列オプション（Int/String options）**  
  `--key=value` 形式で指定します（スペースなし）。例：`--order-by=rand`

- **真偽値オプション（Bool options）**  
  `true/false` を `=on|off|yes|no|1|0` で指定可能。値省略時は `true` と見なされます。

- **フィルター（Filters）**  
  ワイルドカード（`*` = 任意文字列, `?` = 任意1文字）を使用し、カンマ区切りで複数指定可能。引用符やエスケープ（`\`）もサポートされています。

> ※ 全てのオプションはコードからも設定可能です（[`main()` を自作](main.md) した場合）。

---

## クエリフラグ一覧

| フラグ | 説明 |
|-------|------|
| `--help`, `-?`, `-h` | すべてのオプションのヘルプを表示 |
| `--version`, `-v` | doctest のバージョンを表示 |
| `--count`, `-c` | 現在のフィルターで一致するテストケース数を表示 |
| `--list-test-cases`, `-ltc` | 一致するテストケース名を表示 |
| `--list-test-suites`, `-lts` | 一致するテストスイート名を表示 |
| `--list-reporters`, `-lr` | 利用可能なレポーターを表示 |

---

## フィルター・実行制御オプション（Int/String）

| オプション | 内容 |
|------------|------|
| `--test-case=<パターン>` | テストケース名による実行対象のフィルタリング |
| `--test-case-exclude=<パターン>` | 上記の除外版 |
| `--source-file=<パターン>` / `--source-file-exclude=<パターン>` | ファイル名でフィルタ／除外 |
| `--test-suite=<パターン>` / `--test-suite-exclude=<パターン>` | テストスイート名でフィルタ／除外 |
| `--subcase=<パターン>` / `--subcase-exclude=<パターン>` | サブケース名でのフィルタ／除外 |
| `--reporters=<リスト>` | 使用するレポーター（例: `console`, `xml`）|
| `--out=<ファイル名>` | 出力先ファイル |
| `--order-by=<file|suite|name|rand|none>` | テストケースの実行順序（デフォルト：`file`） |
| `--rand-seed=<int>` | ランダム順序用シード値 |
| `--first=<int>` / `--last=<int>` | 実行範囲の先頭・末尾を指定（範囲実行に） |
| `--abort-after=<int>` | 指定数の失敗で実行停止（0 = 無制限） |
| `--subcase-filter-levels=<int>` | ネストされたサブケースのフィルタ適用レベル数 |

---

## フラグ（Bool オプション）

| オプション | 効果 |
|------------|------|
| `--success` | 成功したアサートも出力 |
| `--case-sensitive` | フィルターを大文字小文字区別ありに |
| `--exit` | テスト実行後に明示的に終了（`main()` 自作時） |
| `--duration` | 各テストケースの実行時間を秒で表示 |
| `--minimal` | 失敗したテストのみ出力 |
| `--quiet` | 出力を完全に抑制 |
| `--no-throw` | 例外アサートをスキップ |
| `--no-exitcode` | 常に正常終了コードを返す（失敗しても） |
| `--no-run` | 登録のみでテストは実行しない（初期化のみ） |
| `--no-intro` | 出力の冒頭ヘッダーを省略 |
| `--no-version` | バージョン情報を省略 |
| `--no-colors` | 色出力を無効化 |
| `--force-colors` | tty 非検出時でも色出力を強制 |
| `--no-breaks` | アサート失敗時にデバッガで中断しない |
| `--no-skip` | `skip` デコレータを無効にして常に実行 |
| `--gnu-file-line` | 出力の行番号フォーマットを GNU 形式に（`:n:`） |
| `--no-path-filenames` | 出力からファイルパスを除外 |
| `--no-line-numbers` | 出力の行番号をすべて `0` に置換 |
| `--no-debug-output` | デバッガ接続時のコンソール出力を抑制 |

---

## オプションのプレフィックス変更と無効化

- すべてのオプションは `--dt-` プレフィックス付きでも利用できます（例：`--dt-version`）。
- このプレフィックスは [`DOCTEST_CONFIG_OPTIONS_PREFIX`](configuration.md#doctest_config_options_prefix) で変更可能です。
- プレフィックスなしオプションは [`DOCTEST_CONFIG_NO_UNPREFIXED_OPTIONS`](configuration.md#doctest_config_no_unprefixed_options) で無効化できます。

これにより、doctest を外部プログラムと統合する場合でも、オプション名の競合を避けられます。

---

## オプションフィルタのカスタマイズ例

以下のように独自 `main()` を使い、`--dt-` 付きオプションだけを doctest に渡す例です：

```cpp
#define DOCTEST_CONFIG_NO_UNPREFIXED_OPTIONS
#define DOCTEST_CONFIG_IMPLEMENT
#include "doctest.h"

class dt_removed {
    std::vector<const char*> vec;
public:
    dt_removed(const char** argv_in) {
        for(; *argv_in; ++argv_in)
            if(strncmp(*argv_in, "--dt-", strlen("--dt-")) != 0)
                vec.push_back(*argv_in);
        vec.push_back(NULL);
    }

    int          argc() { return static_cast<int>(vec.size()) - 1; }
    const char** argv() { return &vec[0]; }
};

int program(int argc, const char** argv);

int main(int argc, const char** argv) {
    doctest::Context context(argc, argv);
    int test_result = context.run();

    if(context.shouldExit())
        return test_result;

    dt_removed args(argv);
    int app_result = program(args.argc(), args.argv());

    return test_result + app_result;
}

int program(int argc, const char** argv) {
    printf("アプリケーション側：引数数 = %d\n", argc - 1);
    while(*++argv)
        printf("'%s'\n", *argv);
    return EXIT_SUCCESS;
}
```

### 実行例：

```
program.exe --dt-test-case=math* --my-option -s --dt-no-breaks
```

### 出力：

```
アプリケーション側：引数数 = 2
'--my-option'
'-s'
```

---

[ホームに戻る](readme.md#reference)
