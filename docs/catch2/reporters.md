# レポーター（Reporters）

Catch にはモジュール式のレポートシステムがあり、いくつかの便利なレポーターが標準で同梱されています。  
また、独自のレポーターを作成して使用することもできます。

---

## レポーターの切り替え

使用するレポーターはコマンドラインから簡単に指定できます。  
指定には [`-r` または `--reporter`](command-line.md#choosing-a-reporter-to-use) オプションを使い、レポーター名を続けます。

例：
```
-r xml
```

レポーターを明示的に指定しない場合は、デフォルトで `console` レポーターが使用されます。

---

## 同梱されているレポーター

以下の 4 種類のレポーターがシングルインクルード版に含まれています：

- `console`  
  → 端末幅に合わせて整形されたテキストで出力。対応端末では色付き表示されます。

- `compact`  
  → `console` に似ていますが、出力が最小限に抑えられ、各項目が1行で表示されます。

- `junit`  
  → Ant の [junitreport](http://help.catchsoftware.com/display/ET/JUnit+Format) ターゲットに対応した XML を出力します。  
     CI ツールやビルドシステムとの連携に適しています。  
     ※ 出力はテスト実行完了後に一括で行われます。

- `xml`  
  → Catch 独自の XML 形式で、ストリーミング出力に対応しています。結果が逐次出力されます。

---

## その他のレポーター（必要に応じて追加）

Catch のリポジトリには、特定のビルドシステム向けの追加レポーターが `include/reporters` ディレクトリに用意されています。  
必要であれば、これらを自プロジェクトに `#include` して使用できます。  
この際は、`CATCH_CONFIG_MAIN` または `CATCH_CONFIG_RUNNER` を定義したソースファイルで行ってください。

- `teamcity`  
  → JetBrains [TeamCity](https://www.jetbrains.com/teamcity/) が理解できるストリーミング形式で出力します。  
     TeamCity 上でのビルドに組み込むと、リアルタイムで結果を確認できます。  
     （[コード例](../examples/207-Rpt-TeamCityReporter.cpp) あり）

- `tap`  
  → [Test Anything Protocol (TAP)](https://en.wikipedia.org/wiki/Test_Anything_Protocol) 形式で出力します。

- `automake`  
  → [automake の .trs ファイル](https://www.gnu.org/software/automake/manual/html_node/Log-files-generation-and-test-results-recording.html) に対応する形式で出力します。

- `sonarqube`  
  → [SonarQube Generic Test Data](https://docs.sonarqube.org/latest/analysis/generic-test/) に対応する XML フォーマットを出力します。

---

## 使用可能なレポーターの一覧を確認する

次のコマンドで、使用可能なレポーター一覧を確認できます：

```
--list-reporters
```

---

## 出力先の指定

デフォルトでは、すべてのレポートは `stdout` に出力されます。  
`-o` または `--out` オプションで出力先ファイルにリダイレクトできます。

例：
```
-r xml -o results.xml
```

---

## 独自レポーターの作成

独自のカスタムレポーターを作成して Catch に登録することも可能です。

※ 現在、レポーターのインターフェースは変更が予定されており、まだ正式なドキュメントは提供されていません。

とはいえ、既存の実装を参考にすれば独自レポーターの作成はそれほど難しくありません。  
（今後の変更は主に簡素化の方向で、基底クラスへのフォワードが不要になるなどの軽微な変更が中心です）

---

[ホームへ戻る](Readme.md)
