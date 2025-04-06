# 拡張機能（Extensions）

**doctest** のヘッダは、最適なビルド時間を実現するため、インターフェース部では外部ライブラリや標準ライブラリのヘッダをインクルードしません。そのため、機能が制限される場面もあり、そうした機能を補完するために拡張機能が用意されています。

拡張機能のヘッダは [`doctest/extensions`](../../doctest/extensions) にあり、それぞれに対応したドキュメントがあります。

---

## [Utils（汎用ユーティリティ）](../../doctest/extensions/doctest_util.h)

※現在のところ特記事項はありません。

---

## [MPI を使った分散テスト](../../doctest/extensions/doctest_mpi.h)

提供：Bruno Maugars および Bérenger Berthoul（ONERA）

MPI を使った分散プロセスのテストには、テストフレームワーク側のサポートが必要です。**doctest** は、MPI 通信を利用するコードをテストするためのサポートを `doctest/extensions/doctest_mpi.h` ヘッダで提供しています。

### 使用例

完全なテスト例：[mpi.cpp](../../examples/mpi/mpi.cpp)  
`main()` の構成：[main.cpp](../../examples/mpi/main.cpp)

---

### `MPI_TEST_CASE`

```cpp
#include "doctest/extensions/doctest_mpi.h"

int my_function_to_test(MPI_Comm comm) {
  int rank;
  MPI_Comm_rank(comm, &rank);
  return rank == 0 ? 10 : 11;
}

MPI_TEST_CASE("2プロセスでのテスト", 2) {
  int x = my_function_to_test(test_comm);

  MPI_CHECK(0, x == 10);  // ランク0で x == 10 をチェック
  MPI_CHECK(1, x == 11);  // ランク1で x == 11 をチェック
}
```

- `MPI_TEST_CASE("名前", プロセス数)` は `TEST_CASE` に似ていますが、実行に必要な MPI プロセス数を第2引数で指定します。
- 指定されたプロセス数未満で実行された場合、テストはスキップされます。
- 使用できるオブジェクト：
  - `test_comm`: テスト用 MPI コミュニケータ（`MPI_Comm` 型）
  - `test_rank`: 現在のランク（int）
  - `test_nb_procs`: 使用中のプロセス数（int）

例：

```cpp
MPI_TEST_CASE("例", N) {
  CHECK(test_nb_procs == N);
  MPI_CHECK(i, test_rank == i); // 任意の i < N に対して
}
```

---

### アサーション

`MPI_TEST_CASE` 内では通常のアサーション（`CHECK`, `REQUIRE` など）に加えて、MPI 専用のアサーション（`MPI_CHECK`, `MPI_REQUIRE` など）が使用可能です。

書式：

```cpp
MPI_CHECK(rank, 条件式);
```

---

### `main()` と MPI レポーター

MPI テストは以下のように `mpirun` または `mpiexec` で実行します：

```bash
mpirun -np 2 ./unit_tests
```

セットアップ例：

```cpp
#define DOCTEST_CONFIG_IMPLEMENT
#include "doctest/extensions/doctest_mpi.h"

int main(int argc, char** argv) {
  doctest::mpi_init_thread(argc, argv, MPI_THREAD_MULTIPLE);

  doctest::Context ctx;
  ctx.setOption("reporters", "MpiConsoleReporter");
  ctx.setOption("reporters", "MpiFileReporter");
  ctx.setOption("force-colors", true);
  ctx.applyCommandLine(argc, argv);

  int result = ctx.run();

  doctest::mpi_finalize();
  return result;
}
```

---

### MpiConsoleReporter

- 通常の `console` レポーターに似ていますが、出力はランク0のプロセスに限定されます。
- MPI テストで失敗した場合、どのランクで失敗したかが表示されます。
- 必要なプロセス数に満たない場合、テストはスキップされ、その旨が出力されます。

例：3プロセス必要なテストを2プロセスで実行した場合

```
[doctest] WARNING: Skipped 1 test requiring more than 2 MPI processes to run
```

---

### MpiFileReporter

各プロセスの出力を `doctest_ランク番号.log` というファイルに分けて記録します。  
詳細なデバッグに便利ですが、通常利用には不向きです。

---

### その他のレポーター

`junit`, `xml` などの他のレポーターは MPI 用に対応していません。  
ただし、各プロセスが個別にファイル出力することで代用可能です（集約は未対応）。

---

## 注意

この MPI 機能は「MPI 分散コードの単体テスト用」です。  
**複数の単体テストをプロセスに分散して並列実行する目的ではありません**（それについては [この Python スクリプト](../../examples/range_based_execution.py) を参照）。

---

## 今後の課題（TODO）

- `ConsoleReporter` の内部文字列ストリーム `s` の扱いを柔軟にして共有できるようにする
- `MPI_REQUIRE` や例外対応のテストを追加する
- 自動テスト・CI に対応する
- ビルドシステムとの統合（例：`mpi_doctest` ターゲット）
- レポートの出力形式（console, XML, JUnit）と出力戦略（通常 vs MPI）を分離する仕組みを検討

---

[ホームに戻る](readme.md#reference)
