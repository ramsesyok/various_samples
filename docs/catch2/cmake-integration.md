# CMake との統合

Catch2 は CMake を使って構築されているため、ユーザー向けにいくつかの統合ポイントを提供しています。

1) Catch2 は（名前空間付きの）CMake ターゲットをエクスポートします  
2) Catch2 のリポジトリには、CTest への `TEST_CASE` の自動登録を支援する CMake スクリプトが含まれています

<a id="cmake-target"></a> 

## CMake ターゲット

Catch2 の CMake ビルドは、インターフェースターゲット `Catch2::Catch2` をエクスポートします。このターゲットとリンクすると、必要なインクルードパスや機能が自動的に追加されます。

つまり、Catch2 がシステムにインストールされている場合は、以下のようにすれば十分です：

```cmake
find_package(Catch2 REQUIRED)
target_link_libraries(tests Catch2::Catch2)
```

このターゲットは、Catch2 をサブディレクトリとして使用する場合にも提供されます。たとえば `lib/Catch2` にクローンしている場合：

```cmake
add_subdirectory(lib/Catch2)
target_link_libraries(tests Catch2::Catch2)
```

または、[FetchContent](https://cmake.org/cmake/help/latest/module/FetchContent.html) を使う方法もあります：

```cmake
Include(FetchContent)

FetchContent_Declare(
  Catch2
  GIT_REPOSITORY https://github.com/catchorg/Catch2.git
  GIT_TAG        v2.13.9 # またはそれ以降のリリース
)

FetchContent_MakeAvailable(Catch2)

target_link_libraries(tests Catch2::Catch2)
```

<a id="automatic-test-registration"></a> 

## テストの自動登録

Catch2 のリポジトリには、`TEST_CASE` を CTest に自動登録するための CMake スクリプトが2つ含まれています。これらは `contrib` フォルダ内にあります：

1) `Catch.cmake`（およびその依存 `CatchAddTests.cmake`）  
2) `ParseAndAddCatchTests.cmake`（非推奨）

Catch2 がシステムにインストールされていれば、`find_package(Catch2 REQUIRED)` の後にこれらを使用できます。そうでない場合は、CMake モジュールパスに追加する必要があります。

### `Catch.cmake` と `CatchAddTests.cmake`

`Catch.cmake` は `catch_discover_tests` という関数を提供し、ターゲットからテストを取得します。この関数は、生成された実行ファイルを `--list-test-names-only` フラグ付きで実行し、その出力をパースしてテスト一覧を取得します。

#### 使用例

```cmake
cmake_minimum_required(VERSION 3.5)

project(baz LANGUAGES CXX VERSION 0.0.1)

find_package(Catch2 REQUIRED)
add_executable(foo test.cpp)
target_link_libraries(foo Catch2::Catch2)

include(CTest)
include(Catch)
catch_discover_tests(foo)
```

#### カスタマイズ

`catch_discover_tests` には以下の追加引数を指定できます：

```cmake
catch_discover_tests(target
                     [TEST_SPEC arg1...]
                     [EXTRA_ARGS arg1...]
                     [WORKING_DIRECTORY dir]
                     [TEST_PREFIX prefix]
                     [TEST_SUFFIX suffix]
                     [PROPERTIES name1 value1...]
                     [TEST_LIST var]
                     [REPORTER reporter]
                     [OUTPUT_DIR dir]
                     [OUTPUT_PREFIX prefix]
                     [OUTPUT_SUFFIX suffix]
)
```

- `TEST_SPEC arg1...`  
  Catch 実行ファイルに渡すテスト名やタグ、タグ式のリスト。

- `EXTRA_ARGS arg1...`  
  各テストケースにコマンドライン引数を追加で渡す。

- `WORKING_DIRECTORY dir`  
  テストケースを実行する作業ディレクトリ。指定がない場合はカレントのバイナリディレクトリが使用されます。

- `TEST_PREFIX prefix` / `TEST_SUFFIX suffix`  
  発見されたテスト名にプレフィックスまたはサフィックスを追加。複数の `catch_discover_tests()` 呼び出しでテスト名を区別したい場合に便利です。

- `PROPERTIES name1 value1...`  
  この呼び出しで登録されたすべてのテストに追加プロパティを設定。

- `TEST_LIST var`  
  テスト名の一覧を `<target>_TESTS` の代わりに指定した変数名に格納。CTest でのみ使用可能。

- `REPORTER reporter`  
  実行時に使用する Catch レポーターを指定（例：`--reporter xml`）。

- `OUTPUT_DIR dir`  
  出力先ディレクトリを指定。各テストは `--out dir/<test_name>` で出力されます。並列実行時の競合を避けるため `EXTRA_ARGS --out` より推奨されます。

- `OUTPUT_PREFIX prefix` / `OUTPUT_SUFFIX suffix`  
  `OUTPUT_DIR` と併用。出力ファイル名に接頭辞や接尾辞を追加できます（例：`.xml` 拡張子の付加など）。

### `ParseAndAddCatchTests.cmake`

⚠ このスクリプトは Catch 2.13.4 で[非推奨](https://github.com/catchorg/Catch2/pull/2120)となり、代わりに `catch_discover_tests` の使用が推奨されます。詳細は [#2092](https://github.com/catchorg/Catch2/issues/2092) を参照してください。

このスクリプトは、ターゲットに関連付けられた実装ファイルを解析し、CTest の `add_test` を使って登録します。  
ただし、コメントアウトされたテストも登録されたり、Catch のすべてのマクロに対応しておらず、未対応のマクロを使ったテストは無視されます（しかも警告なし）。

#### 使用例

```cmake
cmake_minimum_required(VERSION 3.5)

project(baz LANGUAGES CXX VERSION 0.0.1)

find_package(Catch2 REQUIRED)
add_executable(foo test.cpp)
target_link_libraries(foo Catch2::Catch2)

include(CTest)
include(ParseAndAddCatchTests)
ParseAndAddCatchTests(foo)
```

#### カスタマイズ

以下の変数を設定することで動作をカスタマイズできます：

- `PARSE_CATCH_TESTS_VERBOSE`  
  `ON` の場合、デバッグメッセージを出力（デフォルト：`OFF`）

- `PARSE_CATCH_TESTS_NO_HIDDEN_TESTS`  
  `ON` の場合、非表示タグ（`[!hide]`, `[.]`, `[.foo]`）付きテストを登録しない（デフォルト：`OFF`）

- `PARSE_CATCH_TESTS_ADD_FIXTURE_IN_TEST_NAME`  
  `ON` の場合、CTest のテスト名にフィクスチャ名を含める（デフォルト：`ON`）

- `PARSE_CATCH_TESTS_ADD_TARGET_IN_TEST_NAME`  
  `ON` の場合、CTest のテスト名にターゲット名を含める（デフォルト：`ON`）

- `PARSE_CATCH_TESTS_ADD_TO_CONFIGURE_DEPENDS`  
  `ON` の場合、テストファイルを `CMAKE_CONFIGURE_DEPENDS` に追加して、ファイル変更時に再構成されるようにする（デフォルト：`OFF`）

さらに、`OptionalCatchTestLauncher` 変数を定義して、テストの実行コマンドをカスタマイズできます。  
例えば、MPI を使って一部のテストを並列実行し、それ以外は通常実行する場合：

```cmake
set(OptionalCatchTestLauncher ${MPIEXEC} ${MPIEXEC_NUMPROC_FLAG} ${NUMPROC})
ParseAndAddCatchTests(mpi_foo)
unset(OptionalCatchTestLauncher)
ParseAndAddCatchTests(bar)
```

## <a id="cmake-project-options"></a> CMake プロジェクトオプション

Catch2 の CMake プロジェクトでは、他プロジェクト向けにいくつかのオプションを提供しています：

- `CATCH_BUILD_TESTING`  
  `ON` の場合、Catch2 の自己テスト（SelfTest）プロジェクトをビルド（デフォルト：`ON`）  
  ※ `BUILD_TESTING` と両方が `ON` である必要があります。どちらかが `OFF` であれば SelfTest はビルドされません。

- `CATCH_BUILD_EXAMPLES`  
  `ON` の場合、Catch2 の使用例をビルド（デフォルト：`OFF`）

- `CATCH_INSTALL_DOCS`  
  `ON` の場合、Catch2 のドキュメントをインストールに含める（デフォルト：`ON`）

- `CATCH_INSTALL_HELPERS`  
  `ON` の場合、Catch2 の contrib フォルダをインストールに含める（デフォルト：`ON`）

- `BUILD_TESTING`  
  `ON` かつサブプロジェクトでない場合に、Catch2 のテストバイナリをビルド（デフォルト：`ON`）

## <a id="installing-catch2-from-git-repository"></a> Git リポジトリから Catch2 をインストール

パッケージマネージャが使用できない場合（例：Ubuntu 16.04 では Catch のバージョンが 1.2.0）、リポジトリから直接インストールすることができます。  
権限があれば、以下のようにインストール可能です：

```
$ git clone https://github.com/catchorg/Catch2.git
$ cd Catch2
$ cmake -Bbuild -H. -DBUILD_TESTING=OFF
$ sudo cmake --build build/ --target install
```

管理者権限がない場合は、CMake の [`CMAKE_INSTALL_PREFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html) を指定し、  
`find_package` 呼び出しにも適切に反映させる必要があります。

## <a id="installing-catch2-from-vcpkg"></a> vcpkg から Catch2 をインストール

あるいは、[vcpkg](https://github.com/microsoft/vcpkg/) という依存関係マネージャを使って Catch2 をビルド・インストールすることも可能です：

```
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
./bootstrap-vcpkg.sh
./vcpkg integrate install
./vcpkg install catch2
```

vcpkg の Catch2 パッケージは Microsoft のチームおよびコミュニティによってメンテナンスされています。  
バージョンが古い場合は、[vcpkg リポジトリ](https://github.com/Microsoft/vcpkg) に Issue または Pull Request を送信してください。

---

[Home](Readme.md)
