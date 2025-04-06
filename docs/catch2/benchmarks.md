# ベンチマークの記述方法

> [Catch 2.9.0 で導入されました](https://github.com/catchorg/Catch2/issues/1616)

⚠️ **デフォルトではベンチマーク機能は無効になっています。使用するには `CATCH_CONFIG_ENABLE_BENCHMARKING` を定義してください。**  
詳細は [コンパイル時設定ドキュメント](configuration.md) を参照してください。

---

ベンチマークを書くのは簡単ではありません。Catch はいくつかの手間を軽減してくれますが、  
正確な測定にはいくつかの前提や理解が必要です。

ここではまず、以降の説明で使用される用語を整理します。

| 用語 | 説明 |
|------|------|
| **ユーザーコード** | 測定対象となるコード。 |
| **ラン (Run)** | ユーザーコードを1回実行すること。 |
| **サンプル (Sample)** | ランを複数回実行した際の測定結果1点。1サンプルに複数回のランが含まれる場合があります。 |

---

## 実行の流れ

Catch によるベンチマーク実行は、以下の3ステップで構成されます：

1. **環境調査 (Environmental probe)**  
   クロックの分解能（精度）や、クロック呼び出しのオーバーヘッドなどを測定します。  
   ※通常は結果に大きな影響を与えません。

2. **実行回数の推定 (Estimation)**  
   ベンチマーク対象コードを数回実行して、1サンプルあたり何回実行すべきかを見積もります。  
   ※この段階でキャッシュのウォームアップも行われます。

3. **測定 (Measurement)**  
   見積もりに基づいてランを実行し、サンプルを収集します。

💡 重要な注意点：  
**ユーザーコードは複数回実行される前提で設計してください。**  
繰り返し実行に適さないコードでは正確な測定ができず、クラッシュや誤った結果を招きます。

---

## ベンチマークの書き方

Catch2 のベンチマークは通常の `TEST_CASE` 内に記述します。

```cpp
std::uint64_t Fibonacci(std::uint64_t number) {
    return number < 2 ? 1 : Fibonacci(number - 1) + Fibonacci(number - 2);
}

TEST_CASE("Fibonacci") {
    CHECK(Fibonacci(0) == 1);
    CHECK(Fibonacci(5) == 8);

    BENCHMARK("Fibonacci 20") {
        return Fibonacci(20);
    };

    BENCHMARK("Fibonacci 25") {
        return Fibonacci(25);
    };
}
```

✅ ポイント：

- `BENCHMARK` はラムダ式として展開されるため、**末尾にセミコロンが必要**です。
- `return` を使って戻り値を返すことで、**コンパイラによる最適化を回避**できます。

---

## 高度なベンチマーク：`BENCHMARK_ADVANCED`

より柔軟な制御が必要な場合は `BENCHMARK_ADVANCED` を使います。  
この形式では、測定前に準備コードを記述できます。

```cpp
BENCHMARK_ADVANCED("advanced")(Catch::Benchmark::Chronometer meter) {
    set_up();
    meter.measure([] { return long_computation(); });
}
```

- `measure` 内のコードのみが測定対象となります。
- `measure([](int i) { ... })` のように、実行回数を渡すこともできます。

```cpp
std::vector<std::string> v(meter.runs());
std::fill(v.begin(), v.end(), test_string());
meter.measure([&v](int i) { in_place_escape(v[i]); });
```

---

### `BENCHMARK("indexed", i)` の形式

簡易な形式でも `i` を指定すれば、`BENCHMARK_ADVANCED` と同様に `i` が渡されます。

```cpp
BENCHMARK("indexed", i) {
    return long_computation(i);
};
```

---

## コンストラクタ・デストラクタの測定

自動変数でオブジェクトを生成すると、**破棄（デストラクション）も含めて測定されてしまう**ため、  
**コンストラクタやデストラクタ単体の時間を測りたい場合は特殊な仕組みが必要**です。

```cpp
BENCHMARK_ADVANCED("construct")(Catch::Benchmark::Chronometer meter) {
    std::vector<Catch::Benchmark::storage_for<std::string>> storage(meter.runs());
    meter.measure([&](int i) { storage[i].construct("thing"); });
};

BENCHMARK_ADVANCED("destroy")(Catch::Benchmark::Chronometer meter) {
    std::vector<Catch::Benchmark::destructable_object<std::string>> storage(meter.runs());
    for (auto&& o : storage)
        o.construct("thing");
    meter.measure([&](int i) { storage[i].destruct(); });
};
```

- `storage_for<T>` は **T型のバッファ** を持つだけで、明示的に `.construct(...)` して初めて構築されます。
- `destructable_object<T>` は明示的な `.destruct()` を必要とし、自動では破棄されません。

---

## 最適化への対処

最適化により、**測定したいコードが消されてしまう**場合があります。

**Catch2 では `return` された値は必ず評価される** ため、`return` を使えば最適化を防げます。

```cpp
BENCHMARK("no return") {
    long_calculation(); // ← 削除される可能性あり
};

BENCHMARK("with return") {
    return long_calculation(); // ← 必ず評価される
};
```

---

### まとめ

- **`BENCHMARK`** で簡単に測定コードを記述できる
- **`BENCHMARK_ADVANCED`** で柔軟な前処理や制御が可能
- **`return`** で最適化を抑止可能
- **構築・破棄時間の測定には `storage_for` / `destructable_object` を使う**

---

📘（このセクションは非公式ベンチマークライブラリ [nonius](https://nonius.io/) のドキュメントを元に適応されています）

---

[ホームへ戻る](Readme.md)
