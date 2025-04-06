まずは通常のCMakeの環境を構築します。

## 3.2.1 ステップ１ 最初のCMake
CMakeでは、```CMakeLists.txt```にソース・ヘッダファイルなどの登録およびビルド設定を記載する。
プロジェクトフォルダ直下に```CMakeLists.txt```を作成する。

```TestTutorial```という名前で、C++17を用いたプロジェクトの作成の場合は、以下のようになる。

```cmake title="CMakeLists.txt"
cmake_minimum_required(VERSION 3.10)
# プロジェクト名:TestTutorial バージョン:0.1.0 言語:C/C++
project(TestTutorial VERSION 0.1.0 LANGUAGES C CXX)

# 設定に実行可能ファイル「TestTutorial」を追加。ソースとしてmain.cppを利用
add_executable(TestTutorial main.cpp)
```

とりあえず、プログラムはなんでも良い

```cpp title="main.cpp"
#include <iostream>

int main(int, char**){
    std::cout << "Hello, from TestTutorial!\n";
}
```

このようなファイル構成になるはず

```
${PROJECT_SOURCE_DIR}
├─ CMakeLists.txt
└─ main.cpp
```

!!! note "Visual Studio Code + CMake Toolsの場合"
    コマンドパレットで```CMake: Quick Start`` を選択することで、本項と同等のファイル生成してくれる。
    
    - 設定項目
        - プロジェクト名　TestTutorial
        - C++プロジェクトの作成
        - Executable 実行可能ファイルの作成
        - その他のオプション 
        - CTest （CTestを選ぶと後の設定がちょっとだけ楽になる。）

## 3.2.2 ステップ２ CMakeによるビルドの確認
### 3.2.2.1 ビルド環境の生成（Out-of-Sourc Build）
CMakeLists.txtのあるフォルダで```cmake -Bbuild . ```のコマンドを実行すると build ディレクトリが作成され、ビルドに関するデータはこちらに格納されるようになる。

```shell title=" Visual Studio の例"
cmake -Bbuild . 

-- Building for: Visual Studio 17 2022
-- Selecting Windows SDK version 10.0.22621.0 to target Windows 10.0.26100.
-- The C compiler identification is MSVC 19.39.33523.0
-- The CXX compiler identification is MSVC 19.39.33523.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: C:/Program Files/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/14.39.33519/bin/Hostx64/x64/cl.exe - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: C:/Program Files/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/14.39.33519/bin/Hostx64/x64/cl.exe - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done (3.6s)
-- Generating done (0.0s)
-- Build files have been written to: C:/Users/Example/Projects/cpptests/build
```

``` title="cmake -Bbuild . 後のフォルダツリー"
├─CMakeLists.txt
├─ main.cpp
├─ build
└─ tests
    ├─ empty_test.cpp
    └─ catch2
        └─catch.hpp
```

!!! info
    環境によってはコンパイラの設定等が必要になる場合があります。
           
### 3.2.2.2 実際にビルドを行う
CMakeLists.txtのあるフォルダで、```cmake --build build ```を実行すると、ビルドが実行され、buildフォルダ以下にビルド結果が生成される。
```shell
> cmake --build build 
MSBuild のバージョン 17.9.8+b34f75857 (.NET Framework)

  1>Checking Build System
  Building Custom Rule C:/Users/Example/Projects/cpptests/CMakeLists.txt
  main.cpp
  TestTutorial.vcxproj -> C:/Users/Example/Projects/cpptests/build/Debug/TestTutorial.exe
  Building Custom Rule C:/Users/Example/Projects/cpptests/CMakeLists.txt
```
