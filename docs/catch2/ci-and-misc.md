# CI とその他の項目

このページでは、Catch を継続的インテグレーション（CI）ビルドシステムにどのように統合できるかについて説明します。  
ビルドシステムには、CMake のような低レベルツールや、Jenkins や TeamCity のようなサーバー上で動作する大規模なシステムが含まれます。このページではその両方を取り扱います。

## 継続的インテグレーションシステム

ビルドサーバーで Catch を使用する上で最も重要な点は、異なるレポーターの使用です。Catch には、主要なビルドサーバーのほとんどに対応できる3つのレポーターが付属しています。必要に応じてさらに統合を深めるためのレポーターも追加可能です（現在は TeamCity、TAP、Automake、SonarQube のレポーターも提供されています）。

これらのうち2つ（XML と JUnit）は組み込みで、もう1つ（TeamCity）は別のヘッダーファイルとして提供されます。今後、他の2つも分離され、Catch のコアをより軽量化する可能性もあります。

### XML レポーター
```-r xml```

XML レポーターは Catch 固有の XML 形式で出力を行います。

この形式の利点は、Catch の動作（特にネストされたセクションのような特殊な機能）に良く対応しており、ストリーミング出力（途中経過を即座に出力）に対応していることです。

欠点は、Catch 専用の形式であるため、既存のビルドサーバーではそのままでは理解できないことです。XSLT を使って HTML などに変換することは可能ですが、その場合ストリーミングの利点は失われます。

### JUnit レポーター
```-r junit```

JUnit レポーターは、JUnit の ANT スキーマに準拠した XML 形式で出力を行います。

この形式の利点は、ほとんどのビルドサーバーがこの形式をサポートしているため、追加の作業なしに取り込めることです。

欠点は、この形式が JUnit の動作を前提として設計されており、Catch の動作とは大きく異なることです。また、すべてのテスト結果を属性として root 要素に書き込む必要があるため、ストリーミング出力ができず、テスト完了後でなければ出力できません。

## その他のレポーター

その他のレポーターは Catch のシングルヘッダー配布には含まれておらず、別途ダウンロードしてインクルードする必要があります。すべてのレポーターは Git リポジトリの `single_include` ディレクトリ内にあり、ファイル名は `catch_reporter_*.hpp` です。  
たとえば TeamCity レポーターを使うには、`single_include/catch_reporter_teamcity.hpp` をダウンロードして Catch 本体の後にインクルードしてください。

```cpp
#define CATCH_CONFIG_MAIN
#include "catch.hpp"
#include "catch_reporter_teamcity.hpp"
```

### TeamCity レポーター
```-r teamcity```

TeamCity レポーターは、TeamCity のサービスメッセージを stdout に書き出します。  
このレポーターを使用するには、追加のヘッダーをインクルードする必要があります。

TeamCity 専用のため、TeamCity を使用している場合に最適なレポーターですが、それ以外の用途には不向きです。ストリーミング形式で出力されますが、TeamCity の UI にはテストスイート（通常はテスト全体）の完了後に結果が表示されます。

### Automake レポーター
```-r automake```

Automake レポーターは、automake が `make check` により期待する [メタタグ](https://www.gnu.org/software/automake/manual/html_node/Log-files-generation-and-test-results-recording.html#Log-files-generation-and-test-results-recording) を出力します。

### TAP（Test Anything Protocol）レポーター
```-r tap```

Catch のテストスイートはインクリメンタルに実行でき、特定のテストだけを実行することもできるため、TAP レポーターではテスト数を最後に出力します。

### SonarQube レポーター
```-r sonarqube```

[SonarQube Generic Test Data](https://docs.sonarqube.org/latest/analysis/generic-test/) に準拠した XML 形式でテストメトリクスを出力します。

## 低レベルツール <a id="low-level-tools"></a>

### プリコンパイル済みヘッダー（PCH）

Catch はプリコンパイル済みヘッダーとしてインクルードするための試験的なサポートを提供していますが、シングルヘッダー形式であるため、ユーザー側でいくつかの作業が必要です：

* プリコンパイル済みヘッダーのファイルには `CATCH_CONFIG_ALL_PARTS` を定義する必要があります。
* 実装ファイルでは以下のように設定する必要があります：
  * `TWOBLUECUBES_SINGLE_INCLUDE_CATCH_HPP_INCLUDED` を `#undef` で解除
  * `CATCH_CONFIG_IMPL_ONLY` を定義
  * `CATCH_CONFIG_MAIN` または `CATCH_CONFIG_RUNNER` を定義
  * その後で再度 `"catch.hpp"` をインクルード

---

### コードカバレッジモジュール（GCOV、LCOV など）

GCOV を使用してコードカバレッジを取得し、CMake と Catch に統合する方法が分からない場合は、次の外部リポジトリに例があります：  
https://github.com/fkromer/catch_cmake_coverage

---

### pkg-config

Catch2 は簡易的な `pkg-config` の統合も提供しており、名前 `catch2` で登録されています。  
そのため、Catch2 をインストールした後は、以下のように `pkg-config` を使用してインクルードパスを取得できます：

```
pkg-config --cflags catch2
```

---

### gdb および lldb 用スクリプト

Catch2 の `contrib` フォルダには、2 つのデバッガ用スクリプトも含まれています：

* `gdbinit`（gdb 用）
* `lldbinit`（lldb 用）

これらをそれぞれのデバッガで読み込むことで、Catch2 の内部処理部分をステップ実行時にスキップできるようになります。

---

## CMake <a id="cmake"></a>

Catch2 の CMake 連携に関する情報は長くなってきたため、専用ページに移動されました。

➡ [CMake 連携ページへ](cmake-integration.md)

---

[ホームに戻る](Readme.md)
