## ビルドシステム

最新の doctest ヘッダファイルは以下から入手できます：  
https://raw.githubusercontent.com/doctest/doctest/master/doctest/doctest.h

`master` を `dev` や特定のタグ（例：`v1.4.8`）に置き換えることで、任意のバージョンを取得できます。

---

### CMake

- doctest をプロジェクト内に1ファイルとして取り込むのが最も簡単な使い方です。  
  以下のような最小限の CMakeLists.txt が動作します：

```cmake
cmake_minimum_required(VERSION 3.0)
project(cmake_test VERSION 0.0.1 LANGUAGES CXX)

# doctest を他のターゲットで使えるようにする
find_package(doctest REQUIRED)

# テスト実行ファイルを作成
add_executable(tests main.cpp)
target_compile_features(tests PRIVATE cxx_std_17)
target_link_libraries(tests PRIVATE doctest::doctest)
```

- GitHub から doctest を自動で取得して外部プロジェクトとして使うこともできます：

```cmake
include(ExternalProject)
find_package(Git REQUIRED)

ExternalProject_Add(
    doctest
    PREFIX ${CMAKE_BINARY_DIR}/doctest
    GIT_REPOSITORY https://github.com/doctest/doctest.git
    TIMEOUT 10
    UPDATE_COMMAND ${GIT_EXECUTABLE} pull
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    INSTALL_COMMAND ""
    LOG_DOWNLOAD ON
)

ExternalProject_Get_Property(doctest source_dir)
set(DOCTEST_INCLUDE_DIR ${source_dir}/doctest CACHE INTERNAL "Path to include folder for doctest")
```

- この変数を使ってインクルードパスを指定できます：

```cmake
# 全体に指定
include_directories(${DOCTEST_INCLUDE_DIR})

# またはターゲットごとに指定
target_include_directories(my_target PUBLIC ${DOCTEST_INCLUDE_DIR})
```

- doctest のリポジトリをサブモジュールなどで持っている場合は、以下のように CMake に組み込めます：

```cmake
add_subdirectory(path/to/doctest)

add_executable(my_tests src_1.cpp src_2.cpp ...)
target_link_libraries(my_tests doctest)
```

- doctest の `CMakeLists.txt` には `install()` が含まれているため、パッケージとしても利用可能です。

- テスト実行ファイルからテストを自動検出して `ctest` に登録するには  
  [`doctest_discover_tests(<target>)`](../../scripts/cmake/doctest.cmake) を利用できます。  
  Catch2 と同様の方法で使用可能です。

---

### パッケージマネージャー

doctest は以下のパッケージマネージャー経由でインストール可能です：

- **vcpkg**  
    ```sh
    git clone https://github.com/Microsoft/vcpkg.git
    cd vcpkg
    ./bootstrap-vcpkg.sh    # Windows は .\bootstrap-vcpkg.bat
    ./vcpkg integrate install
    ./vcpkg install doctest
    ```

    vcpkg における doctest パッケージは Microsoft やコミュニティによって保守されています。  
    バージョンが古い場合は [vcpkg リポジトリ](https://github.com/Microsoft/vcpkg) に issue や PR を送ってください。

- **hunter**
- **conan**
    - https://conan.io/center/doctest  
    - https://github.com/conan-io/conan-center-index/tree/master/recipes/doctest
- **Homebrew**
    ```sh
    brew install doctest
    ```

---

[ホームへ戻る](readme.md#reference)
