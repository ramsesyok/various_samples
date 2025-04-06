CMakeのテスト用のツールとしてCTestがある。
CTestを利用するために、CMakeの基本的な構成空の修正ステップは次の通り。

## 3.3.1 ステップ１ 空のテストコードを作成
```CMakeLists.txt```にテスト用の設定を追加する

```cmake title="CMakeLists.txt"
cmake_minimum_required(VERSION 3.10)
# プロジェクト名:TestTutorial バージョン:0.1.0 言語:C/C++
project(TestTutorial VERSION 0.1.0 LANGUAGES C CXX)

# 設定に実行可能ファイル「TestTutorial」を追加。ソースとしてmain.cppを利用
add_executable(TestTutorial main.cpp)

# テスト用環境の設定
include(CTest)
enable_testing()

## テストコード+ソース全体を実行ファイルとして登録
add_executable(EmptyTest tests/empty_test.cpp )
## テスト用に testsフォルダをインクロードディレクトリに追加
target_include_directories(EmptyTest PUBLIC ${PROJECT_SOURCE_DIR}/tests)
## CTest にEmptyTest の実行方法を登録
add_test(NAME EmptyTest COMMAND EmptyTest)
```
testsフォルダを作成し、空のテストコード```empty_test.cpp```を作成する。

```cpp title="empty_test.cpp"
#define CATCH_CONFIG_MAIN
#include "catch2/catch.hpp"

TEST_CASE("EmptyTest01")
{
    FAIL("Not yet implemented");
}
```

作成すると以下のようなフォルダ・ファイル構成になる。

```
├─CMakeLists.txt
├─ main.cpp
└─ tests
    ├─ empty_test.cpp
    └─ catch2
        └─catch.hpp
```

## 3.3.2 ステップ２ テストコードをビルド
テストコードをビルドする
```shell
> cmake --build build 
MSBuild のバージョン 17.9.8+b34f75857 (.NET Framework)

  1>Checking Build System
  Building Custom Rule C:/Users/Example/Projects/cpptests/CMakeLists.txt
  empty_test.cpp
  EmptyTest.vcxproj -> C:/Users/Example/Projects/cpptests/build/Debug/EmptyTest.exe
  Building Custom Rule C:/Users/Example/Projects/cpptests/CMakeLists.txt
  main.cpp
  TestTutorial.vcxproj -> C:/Users/Example/Projects/cpptests/build/Debug/TestTutorial.exe
  Building Custom Rule C:/Users/Example/Projects/cpptests/CMakeLists.txt
```

!!! info 
    テストコードのビルドが追加されています。
    ```
    empty_test.cpp
    EmptyTest.vcxproj -> C:/Users/Example/Projects/cpptests/build/Debug/EmptyTest.exe
    Building Custom Rule C:/Users/Example/Projects/cpptests/CMakeLists.txt
    ```
## 3.3.3 ステップ３ テストを実行
```ctest --test-dir build -R EmptyTest -C Debug```でテストを実行する。

- --test-dir でbuildディレクトリを指定しています。
- -R は、CMakeLists.txtで指定したテスト名を示します。
- -C Debugは、Visual Studio系の場合、Debug/Relaseのコンフィグレーションが作成されるため、Debugを指定

```shell
> ctest --test-dir build -R EmptyTest -C Debug
Internal ctest changing into directory: C:/Users/Example/Projects/cpptests/build
Test project C:/Users/Example/Projects/cpptests/build
    Start 1: EmptyTest
1/1 Test #1: EmptyTest ........................***Failed    0.03 sec

0% tests passed, 1 tests failed out of 1

Total Test time (real) =   0.04 sec

The following tests FAILED:
          1 - EmptyTest (Failed)
Errors while running CTest
Output from these tests are in: C:/Users/Example/Projects/cpptests/build/Testing/Temporary/LastTest.log        
Use "--rerun-failed --output-on-failure" to re-run the failed cases verbosely.

```
!!! info
    もともと失敗するテストコードのため、テストは失敗している。
    ```shell
    1/1 Test #1: EmptyTest ........................***Failed    0.03 sec
    ```
    成功すると、以下のように出力される。
    ```shell
    Internal ctest changing into directory: C:/Users/Example/Projects/cpptests/build
    Test project C:/Users/Example/Projects/cpptests/build
    Start 1: EmptyTest
    1/1 Test #1: EmptyTest ........................   Passed    0.05 sec
    
    100% tests passed, 0 tests failed out of 1

    Total Test time (real) =   0.05 sec
    ```



