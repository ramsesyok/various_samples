# 自分で `main()` を定義する

Catch の最も簡単な使い方は、Catch 自身に `main()` を定義させ、コマンドライン引数の処理も任せる方法です。

これは、**1つのソースファイルだけで** 以下のように記述することで実現できます：

```cpp
#define CATCH_CONFIG_MAIN
#include "catch.hpp"
```

しかし、時には自分で `main()` を定義したい場合もあるでしょう。その場合は以下のように：

```cpp
#define CATCH_CONFIG_RUNNER
#include "catch.hpp"
```

この定義により、自分で `main()` を書き、Catch を手動で呼び出すことができます。  
これにより、柔軟な制御が可能になります。

⚠️ **注意：`main()` を定義するのは `CATCH_CONFIG_RUNNER` を定義したのと同じソースファイル内でのみ可能です。**

---

## Catch に引数と設定のすべてを任せる

前後に独自の初期化や後処理を入れたいだけであれば、以下のような記述が最も簡単です：

```cpp
#define CATCH_CONFIG_RUNNER
#include "catch.hpp"

int main( int argc, char* argv[] ) {
  // グローバル初期化

  int result = Catch::Session().run( argc, argv );

  // グローバル後処理

  return result;
}
```

---

## 設定を変更する

コマンドライン引数は Catch に任せつつ、一部の設定だけプログラムから上書きしたい場合は次のようにします：

```cpp
#define CATCH_CONFIG_RUNNER
#include "catch.hpp"

int main( int argc, char* argv[] )
{
  Catch::Session session; // インスタンスは1つだけ

  // configData() に書き込めばデフォルト設定を変更できる（推奨）

  int returnCode = session.applyCommandLine( argc, argv );
  if (returnCode != 0) // コマンドラインにエラーがある場合
    return returnCode;

  // configData() や config() を使って、引数で指定された設定をさらに上書きもできる
  // ただし、必要な場合だけにしましょう

  int numFailed = session.run();

  // 一部の Unix 系 OS では戻り値が 8 ビットまでのため、255 に丸められることがあります
  return numFailed;
}
```

💡 `Config` や `ConfigData` の定義を見れば、どのような設定が可能か確認できます。

すべての設定をコードで制御したい場合は、`applyCommandLine()` を呼ばずに省略します。

---

## 独自のコマンドラインオプションを追加する

Catch は [Clara](https://github.com/philsquared/Clara) という強力なコマンドラインパーサーを内部に組み込んでいます。  
Catch2 および Clara 1.0 では、**オプションの合成（composable parsing）** が可能になり、簡単に拡張できます。

```cpp
#define CATCH_CONFIG_RUNNER
#include "catch.hpp"

int main( int argc, char* argv[] )
{
  Catch::Session session;

  int height = 0; // ユーザー定義のコマンドライン変数

  using namespace Catch::clara;
  auto cli =
    session.cli() |
    Opt(height, "height")        // height にバインド
      ["-g"]["--height"]         // 対応するオプション
      ("how high?");             // ヘルプで表示される説明

  session.cli(cli); // Catch に新しい CLI を渡す

  int returnCode = session.applyCommandLine(argc, argv);
  if (returnCode != 0)
    return returnCode;

  if (height > 0)
    std::cout << "height: " << height << std::endl;

  return session.run();
}
```

🔗 詳細は [Clara のドキュメント](https://github.com/philsquared/Clara/blob/master/README.md) を参照してください。

---

## バージョン検出

Catch のバージョンは以下のマクロで取得できます：

- `CATCH_VERSION_MAJOR`
- `CATCH_VERSION_MINOR`
- `CATCH_VERSION_PATCH`

例えば Catch2 のバージョンが `v2.3.4` の場合：

```cpp
CATCH_VERSION_MAJOR → 2  
CATCH_VERSION_MINOR → 3  
CATCH_VERSION_PATCH → 4
```

---

[ホームに戻る](Readme.md)
