# コマンドライン

**目次**<br>
[実行するテストの指定](#specifying-which-tests-to-run)<br>
[使用するレポーターの指定](#choosing-a-reporter-to-use)<br>
[デバッガのブレークポイント](#breaking-into-the-debugger)<br>
[成功したテストの結果表示](#showing-results-for-successful-tests)<br>
[失敗数が一定数を超えた場合の中断](#aborting-after-a-certain-number-of-failures)<br>
[利用可能なテスト、タグ、レポーターの一覧表示](#listing-available-tests-tags-or-reporters)<br>
[出力のファイルへの送信](#sending-output-to-a-file)<br>
[テスト実行の名前指定](#naming-a-test-run)<br>
[例外が投げられることを期待するアサーションのスキップ](#eliding-assertions-expected-to-throw)<br>
[空白文字の可視化](#make-whitespace-visible)<br>
[警告の有効化 ](#warnings)<br>
[実行時間の表示](#reporting-timings)<br>
[テスト名のファイルからの読み込み](#load-test-names-to-run-from-a-file)<br>
[テスト名のみの表示](#list-test-names-only)<br>
[テストケースの実行順序の指定](#order)<br>
[乱数シードの指定](#rng-seed)<br>
[libIdentify に準拠したフレームワーク情報の出力](#libidentify)<br>
[キー入力待ち](#wait-for-keypress)<br>
[ベンチマークのサンプル数指定](#benchmark-samples)<br>
[ブートストラップ用の再サンプル数指定](#benchmark-resamples)<br>
[ブートストラップの信頼区間の指定](#benchmark-confidence-interval)<br>
[統計解析の無効化](#benchmark-no-analysis)<br>
[ウォームアップ時間の指定（ミリ秒）](#benchmark-warmup-time)<br>
[使い方](#usage)<br>
[実行するセクションの指定](run-section)<br>
[ファイル名をタグとして扱う](#filenames-as-tags)<br>
[出力カラー設定の上書き](#use-colour)<br>

Catch はコマンドラインオプションなしでも十分に機能しますが、より詳細な制御が必要な場合のために、以下のオプションが用意されています。  
下記のリンクをクリックすると該当項目にジャンプします。もしくはそのままスクロールしてご覧ください。

以下は「## Specifying which tests to run（実行するテストの指定）」以降の翻訳です：

---

## 実行するテストの指定 { #specifying-which-tests-to-run }

<pre>&lt;test-spec> ...</pre>

テストケース、ワイルドカード付きのテストケース、タグ、およびタグ式はすべて、引数として直接指定できます。タグは角括弧 `[]` で囲むことで識別されます。

テスト仕様（test spec）が何も指定されない場合、すべてのテストケース（ただし「非表示」のテストを除く）が実行されます。  
テストを非表示にするには、タグにピリオド（`.`）で始まる名前を付けるか（例：`[.]`）、または旧式の方法で `[hide]` タグを付けるか、名前を `'./'` で始めます。  
コマンドラインで非表示のテストを指定するには、宣言時にどう指定されていたかに関係なく、`[.]` または `[hide]` を使います。

スペースを含むテスト仕様は、クォートで囲む必要があります。スペースがなければクォートは省略可能です。

ワイルドカード `*` はテストケース名の先頭や末尾につけることで、任意の文字列（空文字列も含む）を代用できます。

テスト仕様は大文字小文字を区別しません。

仕様の前に `exclude:` またはチルダ `~` を付けると、そのパターンは除外条件として扱われます。  
これは、マッチしたテストを除外することを意味し、たとえ以前の仕様で含まれていたとしても除外されます。ただし、後続の inclusion（含める）指定が優先されます。  
inclusion と exclusion の評価は、左から右の順で処理されます。

### テスト仕様の例:

<pre>
thisTestOnly            'thisTestOnly' という名前のテストケースにマッチ
"this test only"        'this test only' という名前のテストケースにマッチ
these*                  'these' で始まるすべてのテストにマッチ
exclude:notThis         'notThis' というテスト以外すべてにマッチ
~notThis                同上（除外指定）
~*private*              'private' を含むすべてのテストを除外
a* ~ab* abc             'a' で始まるすべてのテストから、'ab' で始まるものを除き、'abc' を明示的に含む
-# [#somefile]          'somefile.cpp' というファイル内のすべてのテストにマッチ
</pre>

角括弧で囲まれた名前はタグとして解釈されます。  
タグを連続で記述すると AND 条件、カンマ区切りで記述すると OR 条件として評価されます。  
例:

<pre>[one][two],[three]</pre>

これは `[one]` および `[two]` の両方が付けられたテスト、または `[three]` が付けられたテストにマッチします。

カンマや角括弧 `[` などの特殊文字を含むテスト名をコマンドラインで指定するには、バックスラッシュ `\` を使用してエスケープします。  
バックスラッシュ自体も `\\` のようにエスケープが必要です。


## 使用するレポーターの指定 { #choosing-a-reporter-to-use }
<pre>-r, --reporter &lt;reporter&gt;</pre>

レポーターとは、テストの実行結果をフォーマットおよび構造化し、必要に応じて結果を要約するオブジェクトです。デフォルトでは、IDE に適したテキスト出力を行うコンソールレポーターが使用されます。Catch にはいくつかの代替レポーターが同梱されていますが、独自にクライアントコードで追加することも可能です。  
同梱されているレポーターは以下の通りです：

<pre>-r console  
-r compact  
-r xml  
-r junit  
</pre>

JUnit レポーターは、JUnit XML Report ANT タスクの構造に準拠した XML フォーマットで出力します。この形式は、Hudson のような継続的インテグレーション（CI）サーバーを含む多くのサードパーティーツールで利用できます。  
特に必要でなければ、標準の XML レポーターの使用が推奨されます。これはストリーミング形式であり、逐次結果が出力されるのに対し、Junit レポーターはすべてのテスト結果を収集してから出力する必要があります（ルートノードの属性に全体の結果を記録するため）。

## デバッガのブレークポイント { #breaking-into-the-debugger }
<pre>-b, --break</pre>

ほとんどのデバッガでは、Catch2 はテストの失敗時に自動的にブレーク（中断）できます。これにより、失敗時のテストの状態を直接確認することができます。

## 成功したテストの結果表示  { #showing-results-for-successful-tests }
<pre>-s, --success</pre>

通常は失敗したテスト結果だけを表示しますが、すべての出力（成功・失敗の両方）を確認したい場合にこのオプションが便利です。特に、新しく追加したテストが最初から正しく動作しているか確認したいときに役立ちます。  
このオプションを指定すると、成功したテストも含めてすべての結果が表示されます。ただし、レポーターの種類によってこのオプションの扱いは異なる場合があります。たとえば、Junit レポーターはこのオプションに関係なくすべての結果をログに記録します。

## 失敗数が一定数を超えた場合の中断 { #aborting-after-a-certain-number-of-failures }
<pre>-a, --abort  
-x, --abortx [&lt;failure threshold&gt;]  
</pre>

```REQUIRE``` アサーションが失敗すると、そのテストケースは中断されますが、他のテストケースの実行は継続されます。  
```CHECK``` アサーションが失敗した場合は、テストケース自体も中断されません。

しかし、多くの失敗が連続すると出力が多くなりすぎるため、最初の数個だけを確認したいこともあります。  
```-a``` または ```--abort``` を指定すると、最初のアサーション失敗でテスト実行全体が中断されます。  
```-x``` または ```--abortx``` に数値を続けて指定すると、その数だけ失敗した後に中断されます。

## 利用可能なテスト、タグ、レポーターの一覧表示 { #listing-available-tests-tags-or-reporters }
<pre>-l, --list-tests  
-t, --list-tags  
--list-reporters  
</pre>

```-l``` または ```--list-tests``` を使用すると、すべての登録済みテストと、それに付けられているタグが一覧表示されます。  
テスト指定（test-spec）が一つ以上指定されている場合は、それに一致するテストのみが表示されます。

```-t``` または ```--list-tags``` を使用すると、利用可能なすべてのタグが表示され、それぞれに一致するテストケースの数も確認できます。こちらもテスト指定がある場合は、それに基づいて絞り込まれます。

```--list-reporters``` を使用すると、利用可能なレポーターの一覧が表示されます。

## 出力をファイルに送る  { #sending-output-to-a-file }
<pre>-o, --out &lt;filename&gt;</pre>

このオプションを使用すると、すべての出力をファイルに送ることができます。デフォルトでは出力は標準出力（stdout）に送られます（なお、テストケース内での stdout や stderr の使用はリダイレクトされ、レポートに含まれるため、実質的には stderr も stdout に出力されます）。

## テスト実行に名前を付ける  { #naming-a-test-run }

<pre>-n, --name &lt;name for test run&gt;</pre>

名前を指定すると、レポーターがテスト実行全体に名前を付けて出力します。たとえば、出力をファイルに送る場合など、異なる Catch 実行ファイルや異なるオプションで実行した結果を区別するのに便利です。名前が指定されない場合、デフォルトで実行ファイルの名前が使用されます。

## 例外を投げることを期待するアサーションの省略  { #eliding-assertions-expected-to-throw }
<pre>-e, --nothrow</pre>

例外が投げられることを確認するアサーション（例：<code>REQUIRE_THROWS</code>）をすべてスキップします。

これは、一部のデバッガ環境では例外がスローされるたびにブレークする場合があるため、ノイズを避ける目的で役立ちます（通常はハンドルされた例外にはブレークしないように設定可能ですが、予期しない挙動を追跡する場合に有効です）。

コード内で例外をスローしてキャッチするような処理がある場合（アサーションの外で発生することもある）、<code>[!throws]</code>タグを使ってテストケース全体をスキップすることもできます。

このオプションを有効にすると、例外チェック用アサーションはスキップされるため、出力のノイズを減らすことができます。ただし、この影響がその後のテストに影響を与えないよう注意してください。

## 空白文字を可視化する  { #make-whitespace-visible }
<pre>-i, --invisibles</pre>

文字列の比較で、空白（特に前後の空白）の違いによって失敗した場合、その差を確認するのが困難なことがあります。このオプションを使用すると、タブや改行などの制御文字を <code>\t</code> や <code>\n</code> のように可視化して表示します。

## 警告の有効化 { #warnings }
<pre>-w, --warn &lt;warning name&gt;</pre>

テストの疑わしい状態に関する警告出力を有効にします。現在サポートされている警告は次の2つです：

```
NoAssertions   // テストケースまたは末端のセクションでアサーション（例：REQUIRE）が1つもなかった場合に失敗とする
NoTests        // 実行されたテストケースが1つもなかった場合、非ゼロの終了コードを返す（レポーターの noMatchingTestCases メソッドも呼ばれる）
```

## 実行時間の表示  { #reporting-timings }
<pre>-d, --durations &lt;yes/no&gt;</pre>

<code>yes</code> を指定すると、Catch は各テストケースの実行時間をミリ秒単位で表示します。この表示はテストが成功した場合でも失敗した場合でも行われます。なお、一部のレポーター（例：JUnit）は、このオプションに関係なく常に実行時間を表示します。

<pre>-D, --min-duration &lt;value&gt;</pre>

> `--min-duration` は Catch 2.13.0 にて [追加されました](https://github.com/catchorg/Catch2/pull/1910)

このオプションを設定すると、指定した秒数より長くかかったテストケースの実行時間のみが表示されます。このオプションは <code>-d yes</code> や <code>-d no</code> の指定によって上書きされるため、すべての時間を表示するか全く表示しないかのいずれかになります。


## ファイルから実行するテスト名を読み込む  { #load-test-names-to-run-from-a-file }
<pre>-f, --input-file &lt;filename&gt;</pre>

1行に1つのテストケース名を記述したファイルの名前を指定します。空行は無視され、コメント文字 `#` 以降の内容も無視されます。

このファイルの初期バージョンを作成する便利な方法としては、<a href="#list-test-names-only">list-test-names-only</a> オプションを使って出力を保存することです。その後、手動でテストの順序や実行対象を調整できます。

## テスト名のみを出力する  { #list-test-names-only }
<pre>--list-test-names-only</pre>

すべてのテストをインデントなしで1行ずつ一覧表示します。これにより、ファイルに保存して <a href="#input-file">`-f` や `--input-file`</a> オプションで再利用しやすくなります。

## テスト実行順を指定する  { #order }
<pre>--order &lt;decl|lex|rand&gt;</pre>

テストケースの実行順は、次の3通りから選択できます：

### decl  
定義順（これがデフォルト）。  
同じ翻訳単位（Translation Unit）内のテストは、記述された順に並びます。異なる翻訳単位間の順序は、リンクの順序など実装依存となります。

### lex  
辞書順（名前順）。  
テストケース名でソートされ、タグは無視されます。

### rand  
ランダム順。  
Catch2 の乱数シード（[`--rng-seed`](#rng-seed) 参照）に基づいて並びます。部分実行しても相対順序が保たれるため、サブセットの実行でも順序が変わらないという特徴があります。

> この部分安定性は Catch2 v2.12.0 で導入されました。

## 乱数シードを指定する  { #rng-seed }
<pre>--rng-seed &lt;'time'|number&gt;</pre>

乱数生成器のシードを `std::srand()` を使って設定します。  
数値を指定するとその値がシードとして使用され、ランダム順でも再現可能になります。  
代わりに `time` を指定すると、`std::time(0)` の結果が使われ、毎回異なる順序になります。環境によっては `'time'` を `"time"` のようにダブルクオートで囲む必要があります。

いずれの場合も、使用されたシード値は Catch の出力に表示されるため、ランダム順で何か問題があった場合も再現が可能です。

## libIdentify 標準に従ってフレームワークとバージョンを識別する  { #libidentify }
<pre>--libidentify</pre>

詳細および例については、[LibIdentify リポジトリ](https://github.com/janwilmans/LibIdentify) をご覧ください。

## キー入力を待機する  { #wait-for-keypress }
<pre>--wait-for-keypress &lt;never|start|exit|both&gt;</pre>

実行前、実行後、またはその両方で Enter キーの入力を待ちます。これは、テスト実行の開始や結果確認の前に一時停止したい場合に便利です。

## ベンチマークのサンプル数を指定する  { #benchmark-samples }
<pre>--benchmark-samples &lt;# of samples&gt;</pre>

> [Catch 2.9.0](https://github.com/catchorg/Catch2/issues/1616) にて導入されました。

ベンチマークを実行する際に収集する「サンプル」の数を指定します。サンプルとは、一定回数のユーザーコードの実行時間を測定して得られた1つのデータ点です。各サンプル内での実行回数は時計の精度によって決まり、サンプル数とは独立です。デフォルトは 100 です。

## ブートストラップでの再サンプリング数を指定する  { #benchmark-resamples }
<pre>--benchmark-resamples &lt;# of resamples&gt;</pre>

> Catch 2.9.0 にて導入されました。

ベンチマーク測定後、統計的な [ブートストラップ](http://en.wikipedia.org/wiki/Bootstrapping_%28statistics%29) が行われます。その際の再サンプル数を指定できます。デフォルトは 100000 回です。  
ブートストラップにより、平均や標準偏差の推定値（および上下限）が得られます。信頼区間の指定も可能で、デフォルトは 95% です。

## ブートストラップの信頼区間を指定する  { #benchmark-confidence-interval }
<pre>--benchmark-confidence-interval &lt;confidence-interval&gt;</pre>

> Catch 2.9.0 にて導入されました。

ブートストラップで平均や標準偏差の上下限を計算するための信頼区間を設定します。  
0〜1 の間の値で指定し、デフォルトは 0.95（95%）です。

## ベンチマークサンプルの統計解析を無効化する  { #benchmark-no-analysis }
<pre>--benchmark-no-analysis</pre>

> Catch 2.9.0 にて導入されました。

このフラグを指定すると、ブートストラップやその他の統計解析を行わず、単純に測定された平均値のみが報告されます。

## 各テストのウォームアップ時間をミリ秒単位で指定する  { #benchmark-warmup-time }
<pre>--benchmark-warmup-time</pre>

> [Catch 2.11.2](https://github.com/catchorg/Catch2/pull/1844) にて導入されました。

各ベンチマークのウォームアップに費やす時間をミリ秒単位で指定します。

## 使用方法の表示  { #usage }
<pre>-h, -?, --help</pre>

コマンドライン引数の一覧と使用方法を標準出力に表示します。

## 実行するセクションを指定する  { #run-section }
<pre>-c, --section &lt;section name&gt;</pre>

テストケースの中の特定セクションだけを実行したい場合、このオプションを使って指定します。  
ネストしたセクションを指定する場合は、より深い階層ごとにこのオプションを複数指定します。

例：

<pre>
TEST_CASE( "Test" ) {
  SECTION( "sa" ) {
    SECTION( "sb" ) {
      /*...*/
    }
    SECTION( "sc" ) {
      /*...*/
    }
  }
  SECTION( "sd" ) {
    /*...*/
  }
}
</pre>

`sb` のみを実行したい場合：
<pre>./MyExe Test -c sa -c sb</pre>

`sd` のみを実行：
<pre>./MyExe Test -c sd</pre>

`sa` 以下すべて（`sb` と `sc` 含む）を実行：
<pre>./MyExe Test -c sa</pre>

注意点：
- スキップされたセクションの外側にあるコード（例えば、TEST_CASE 内の初期化処理）は実行されます。  
- 現時点では、セクション名にワイルドカードは使用できません。  
- テストケースを指定せずにセクションを指定した場合、すべてのテストケースが対象になります（ただしマッチするセクションのみが実行されます）。

## ファイル名をタグとして使用する  { #filenames-as-tags }
<pre>-#, --filenames-as-tags</pre>

このオプションを使用すると、各テストにはそのファイル名（拡張子なし）に `#` を付けたタグが追加されます。

例えば、ファイル `~\Dev\MyProject\Ferrets.cpp` 内のテストは `[#Ferrets]` というタグを持ちます。

## 出力のカラー表示の上書き  { #use-colour }
<pre>--use-colour &lt;yes|no|auto&gt;</pre>

Catch は端末出力時にカラー表示を行いますが、出力先がパイプであると判断されるとカラーは無効になります。これは、自動処理との干渉を避けるためです。

`--use-colour yes` で強制的にカラー表示、`--use-colour no` でカラーを無効化できます。  
デフォルトは `--use-colour auto` です。

---

[ホームに戻る](Readme.md)
