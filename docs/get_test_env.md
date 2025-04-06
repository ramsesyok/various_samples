## 3.1.1 使用するオープンソース
- cmake
    - CMakeは、C++のプロジェクトを簡単かつ柔軟にビルドするためのツール。プラットフォームやコンパイラに依存しないビルド設定が可能で、大規模な開発やチーム開発にも適している。MakefileやVisual Studioのプロジェクトファイルなどを自動生成できる。

- catch2
    - Catch2は、C++向けの多様なパラダイムに対応したテストフレームワークであり、Objective-C（および場合によってはC）もサポートしている。主にシングルヘッダファイルとして配布されているが、特定の拡張機能では追加のヘッダファイルが必要になる場合もある。
    - Catch2 v3 から、シングルヘッダファイルとしての提供から、ライブラリとしての提供に変更された。
    
        !!! info
            シングルヘッダファイルによるライブラリは、環境構築の容易性。依存の少なさなどメリットがあるいっぽう。
            規模が大きくなった際に、ビルドが遅くなるなどのデメリットがある。


## 3.1.2 cmakeの環境を作る
本家サイト[:simple-cmake: CMake - Upgrade Your Software Build System](https://cmake.org/)の[ダウンロードページ](https://cmake.org/download/)から表のいずれかをダウンロードする。

| Platform | Files |
| -------- | ----- |
| Windows x64 Installer: | [cmake-4.0.0-windows-x86_64.msi](https://github.com/Kitware/CMake/releases/download/v4.0.0/cmake-4.0.0-windows-x86_64.msi) |
| Windows x64 ZIP | [cmake-4.0.0-windows-x86_64.zip](https://github.com/Kitware/CMake/releases/download/v4.0.0/cmake-4.0.0-windows-x86_64.zip) |
| Linux x86_64 | [cmake-4.0.0-linux-x86_64.sh](https://github.com/Kitware/CMake/releases/download/v4.0.0/cmake-4.0.0-linux-x86_64.sh) |
| | [cmake-4.0.0-linux-x86_64.tar.gz](https://github.com/Kitware/CMake/releases/download/v4.0.0/cmake-4.0.0-linux-x86_64.tar.gz) |


```Windows x64 Installer``` と ```cmake-4.0.0-linux-x86_64.sh``` は、インストーラ形式になっており、手間がなく簡単にインストールができる。

```Windows x64 ZIP```と ```cmake-4.0.0-linux-x86_64.tar.gz``` は、必要ファイルが圧縮されただけの形式のため、インストール時の権限変更ができない場合や、開発端末に余計な設定を行いたくない場合に用いる。

#### Visual Studio Codeの拡張機能
Visual Studio Codeを利用している場合、拡張機能 [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)を導入すると、いろいろ使いやすくなる。


### 3.1.3 catch2の導入
- [:simple-github: catchorg/Catch2(v2.13.10)](https://github.com/catchorg/Catch2/tree/v2.13.10)から取得できる。
- [:material-file-download: ヘッダファイルのRawファイルへのリンク](https://raw.githubusercontent.com/catchorg/Catch2/v2.13.10/single_include/catch2/catch.hpp) より入手することもできる。
- このチュートリアルサイトからも取得できるのできる。[:material-download-box:](./examples/catch.hpp)

#### ダウンロードしたファイルの扱い
- インクルードパスに保存されていれば、特に規定がない。
