# イベントリスナー（Event Listeners）

`Listener`（リスナー）は、Catch に登録できるクラスで、テスト実行中に発生するイベント（テストケースの開始・終了など）をリアルタイムで受け取ることができます。

リスナーは実は `Reporter` の一種ですが、いくつかの違いがあります：

1. **コード内で登録すれば自動で有効**：コマンドラインで明示的に指定する必要はありません。
2. **レポーターと併用可能**：レポーターの直前に呼び出され、複数のリスナーを登録できます。
3. **`Catch::TestEventListenerBase` を継承**：すべてのイベントに対するデフォルトの空実装があるため、関心のあるイベントだけを実装できます。
4. **登録には `CATCH_REGISTER_LISTENER` マクロを使用**します。

---

## リスナーの実装方法

`Catch::TestEventListenerBase` を継承し、必要なイベントだけをオーバーライドします。  
定義は以下のいずれかのソースファイル内に記述します：

- `CATCH_CONFIG_MAIN` または `CATCH_CONFIG_RUNNER` を定義したメインソースファイル  
- `CATCH_CONFIG_EXTERNAL_INTERFACES` を定義したファイル

登録には `CATCH_REGISTER_LISTENER` マクロを使います。

### 実装例（[完全なコードはこちら](../examples/210-Evt-EventListeners.cpp)）

```cpp
#define CATCH_CONFIG_MAIN
#include "catch.hpp"

struct MyListener : Catch::TestEventListenerBase {

    using TestEventListenerBase::TestEventListenerBase; // コンストラクタの継承

    void testCaseStarting( Catch::TestCaseInfo const& testInfo ) override {
        // テストケースの実行前にセットアップ処理を行う
    }
    
    void testCaseEnded( Catch::TestCaseStats const& testCaseStats ) override {
        // テストケースの実行後にクリーンアップ処理を行う
    }
};
CATCH_REGISTER_LISTENER( MyListener )
```

⚠️ **リスナーの中でアサーションマクロ（`REQUIRE` など）を使用してはいけません！**

---

## フック可能なイベント一覧

リスナーでオーバーライド可能なイベントメソッドは以下のとおりです：

```cpp
// テスト全体の開始と終了
virtual void testRunStarting( TestRunInfo const& testRunInfo );
virtual void testRunEnded( TestRunStats const& testRunStats );

// テストケースの開始と終了
virtual void testCaseStarting( TestCaseInfo const& testInfo );
virtual void testCaseEnded( TestCaseStats const& testCaseStats );

// セクションの開始と終了
virtual void sectionStarting( SectionInfo const& sectionInfo );
virtual void sectionEnded( SectionStats const& sectionStats );

// アサーションの開始と終了
virtual void assertionStarting( AssertionInfo const& assertionInfo );
virtual bool assertionEnded( AssertionStats const& assertionStats );

// スキップされたテスト（たとえば非表示の場合）
virtual void skipTest( TestCaseInfo const& testInfo );
```

引数として渡される構造体（`TestCaseInfo`, `SectionStats` など）には、テスト名や統計情報など、詳細なデータが含まれています。  
具体的なフィールドは Catch のソースコードを確認してください。

---

[ホームへ戻る](Readme.md)
