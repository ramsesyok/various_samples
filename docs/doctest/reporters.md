# レポーター（Reporters）

**doctest** には、モジュール式のレポーター／リスナー機構があります。ユーザーは独自のレポーターを作成し、登録することができます。また、イベントの「リスニング」にも使えます。

現在登録されているレポーターは `--list-reporters` で一覧表示できます。

## 標準で用意されているレポーター

- `console`: ストリーミング出力。ターミナルにカラー表示付きで出力。
- `xml`: ストリーミング出力。doctest 独自形式の XML を出力。
- `junit`: バッファリング出力。JUnit 互換の XML を出力（[関連 Issue 1](https://github.com/doctest/doctest/issues/318)、[Issue 2](https://github.com/doctest/doctest/issues/376) 参照）。

> **ストリーミング**： テスト実行中に随時出力される形式。  
> **バッファリング**： テスト実行完了後にまとめて出力される形式。

出力はデフォルトで `stdout` に書き出されますが、`--out=<filename>` の [コマンドラインオプション](commandline.md) を使ってファイル出力も可能です。

---

## 独自のレポーターを定義する例

```cpp
#include <doctest/doctest.h>
#include <mutex>

using namespace doctest;

struct MyXmlReporter : public IReporter {
    std::ostream&         stdout_stream;
    const ContextOptions& opt;
    const TestCaseData*   tc;
    std::mutex            mutex;

    MyXmlReporter(const ContextOptions& in)
        : stdout_stream(*in.cout), opt(in) {}

    void report_query(const QueryData&) override {}
    void test_run_start() override {}
    void test_run_end(const TestRunStats&) override {}
    void test_case_start(const TestCaseData& in) override { tc = &in; }
    void test_case_reenter(const TestCaseData&) override {}
    void test_case_end(const CurrentTestCaseStats&) override {}
    void test_case_exception(const TestCaseException&) override {}

    void subcase_start(const SubcaseSignature&) override {
        std::lock_guard<std::mutex> lock(mutex);
    }
    void subcase_end() override {
        std::lock_guard<std::mutex> lock(mutex);
    }

    void log_assert(const AssertData& in) override {
        if (!in.m_failed && !opt.success) return;
        std::lock_guard<std::mutex> lock(mutex);
        // 独自出力処理
    }

    void log_message(const MessageData&) override {
        std::lock_guard<std::mutex> lock(mutex);
        // メッセージ出力処理
    }

    void test_case_skipped(const TestCaseData&) override {}
};
```

登録するには以下のようにします：

```cpp
REGISTER_REPORTER("my_xml", 1, MyXmlReporter);     // 実行時に選択可能なレポーター
REGISTER_LISTENER("my_listener", 1, MyXmlReporter); // 常に実行されるリスナー
```

- `"my_xml"`：レポーター名
- `1`：優先度（小さいほど先に実行。標準の console/xml は `0`）

---

## 実行時に複数のレポーターを使う方法

```bash
--reporters=myReporter,xml
```

複数指定することで、それぞれのレポーターが優先度順に実行されます。

---

## リスナーについて

`REGISTER_LISTENER` で登録されたリスナーは、**すべての実行時に常に呼び出され**ます。  
コマンドラインでの指定やフィルタリングは不要／不可です。ロギングや統計など、グローバルな監視用途に便利です。

---

## 注意点と参考情報

- 複数スレッドでのログ出力に対応するため、`std::mutex` で排他制御するのが推奨されます。
- カスタムレポーターを作成する際は、上記例や doctest に同梱されている標準レポーターのコードを参考にしてください。
- [レポーターとリスナーの全機能例](../../examples/all_features/reporters_and_listeners.cpp)も参照してください。

---

[ホームに戻る](readme.md#reference)
