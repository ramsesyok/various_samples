cmake_minimum_required(VERSION 3.10)
project(UnitTestTutorial LANGUAGES CXX)

# C++17 を使用する場合の設定（必要に応じて変更）
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# src 配下のファイルを全部対象にしておく
file(GLOB_RECURSE SOURCES src/*.cpp)

# include ディレクトリにソースとテストの双方を追加
include_directories(${PROJECT_SOURCE_DIR}/include ${PROJECT_SOURCE_DIR}/tests)

# テスト用実行ファイルの追加
enable_testing()

## テストコード+ソース全体を実行ファイルとして登録
add_executable(EmptyTest tests/EmptyTest.cpp ${SOURCES})
## CTest に登録
add_test(NAME EmptyTest COMMAND EmptyTest)

