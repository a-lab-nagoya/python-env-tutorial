---
marp: true
theme: gaia
paginate: true
backgroundImage: url('https://marp.app/assets/hero-background.jpg')
---

# Construction of Python environments on macOS

![bg right:30% 75%](https://openastronomy.org/pyastro/images/pyastro_logo_150px.png)

- 2020-06-17 / Python tutorials
- Presented by Akio Taniguchi (PD)

---

### Preparation

当日までに、以下の準備と理解をお願いします。

- [Homebrew](https://brew.sh)によるPythonのインストール
    - `$ brew install python`
- Bash（zsh）の最低限のコマンドの理解
    - `cd`, `mv`, `rm`, `mkdir`, `ls`あたりが使えればOKです
- Bash（zsh）の環境変数`PATH`の理解
    - いわゆる「パスを通す」とは何かが分かればOKです
    - 参考：[PATHを通すとは？ (Mac OS X) - Qiita](https://qiita.com/soarflat/items/09be6ab9cd91d366bf71)

---

### Contents

1. 環境構築とは何か
1. Pythonのディレクトリ構造を理解する
    - 実行プログラム・標準ライブラリ・外部ライブラリ
1. 標準的なPythonの環境構築
    - 仮想環境とは何か・なぜ必要か
    - 標準ライブラリ（venv）を使った環境構築
1. ツールを使ったPythonの環境構築
    - Pipenv / PoetryによるPython tutorials対応の環境構築

---

### 環境構築とは何か

- 環境構築
    - とあるプロジェクトに必要な実行プログラム・ライブラリを、**他のプロジェクトとは独立に**ローカルにインストールすること
    - エディタ・linter（構文チェック）・formatter（自動整形）などプログラミングを支援する環境を整えること（今回の範囲外）
- Pythonの環境構築
    - 実行プログラム・標準ライブラリ → Homebrew
    - 外部ライブラリ → venv, pip, あるいはパッケージ管理ツール

---

### Pythonのディレクトリ構造

- 実行プログラム（一部抜粋）
    - `/path/to/bin/`配下に存在する
        - python(3)：Pythonスクリプトを実行するプログラム
        - pip(3)：外部ライブラリをインストールするためのプログラム
- 標準ライブラリ：Pythonに付属のライブラリ群
    - `/path/to/lib/python3.x/`配下に存在する
- 外部ライブラリ：ユーザがpipでインストールしたライブラリ
    - `/path/to/lib/python3.x/site-packages/`配下に保存される

---

### Pythonのディレクトリ構造

これらのインストール先は以下のように確認できる

```shell
$ which python3
/usr/local/bin/python3

$ which pip3
/usr/local/bin/pip3

$ python3 -m site
sys.path = [
    '/usr/local/Cellar/python',
    '/usr/local/Cellar/python/3.7.7/Frameworks/Python.framework/Versions/3.7/lib/python37.zip',
    '/usr/local/Cellar/python/3.7.7/Frameworks/Python.framework/Versions/3.7/lib/python3.7',
    '/usr/local/Cellar/python/3.7.7/Frameworks/Python.framework/Versions/3.7/lib/python3.7/lib-dynload',
    '/usr/local/lib/python3.7/site-packages',
    '/usr/local/Cellar/python/3.7.7/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages',
]
```

---

### 標準的なPythonの環境構築｜単純な環境構築

- 単純な環境構築
    1. Python（とpip）をHomebrewでインストールする
    1. pipを使ってsite-packagesに直に外部ライブラリを追加する
- 利点と欠点
    - :slightly_smiling_face: シンプルで分かりやすい（共通Python環境としてはアリ）
    - :slightly_frowning_face: 異なるバージョンのライブラリが共存できない
        - 必要なライブラリが異なるプロジェクトの両立が難しい
        - ライブラリ自身も他のライブラリに依存していることもある
    - :slightly_frowning_face: 構築に失敗した場合全てのプロジェクトに影響が出る

---

### 標準的なPythonの環境構築｜仮想環境による構築

- 仮想環境を使った環境構築（推奨）
    1. Python（とpip）をHomebrewでインストールする
    1. **プロジェクトごとに仮想環境を作成する**
    1. **仮想環境の**site-packagesに外部ライブラリを追加する
- 利点と欠点
    - :slightly_smiling_face: プロジェクトごとに必要なライブラリを用意できる
    - :slightly_smiling_face: プロジェクトに必要なライブラリ**だけ**を管理できる
    - :slightly_smiling_face: 構築に失敗した場合の対処が楽（仮想環境を作り直すだけ）
    - :slightly_frowning_face: 仮想環境に出入りするのが若干面倒（→ツールを使う）

---

### 標準的なPythonの環境構築｜仮想環境による構築

- 仮想環境（virtual environment）とは
    - プロジェクト（のディレクトリ）ごとに独自の実行プログラムとsite-packagesを作成し、利用可能にするためのPythonの仕組み
- 仮想環境の作成
    - 標準ライブラリ`venv`を使用する
- 仮想環境の使用
    - 有効化（activate）：環境変数`PATH`を一時的に書き換え、実行プログラムをシェルから優先的に利用可能にする
    - 無効化（deactivate）：環境変数`PATH`を元に戻す

---

![bg fit](https://ebi-works.com/wp-content/uploads/2019/12/pyenv-venv3-1-1024x494.png)

<!-- _footer: https://ebi-works.com/pyenv-renv -->

---

### 標準的なPythonの環境構築｜仮想環境による構築

- 仮想環境の作成
    - 仮想環境に必要な諸々はプロジェクト配下の`.venv`ディレクトリに格納される
    - ディレクトリ名は何でも良いが、慣例的に`.venv`が使われる

```shell
$ mkdir /path/to/project # プロジェクト用ディレクトリを作成
$ cd /path/to/project

$ python3 -m venv .venv # プロジェクト配下に仮想環境を作成
$ ls -a
.  ..  .venv
```

---

### 標準的なPythonの環境構築｜仮想環境による構築

`.venv`には実行プログラム（のリンク）とsite-packagesが存在する

```
$ tree -L 3 .venv
.venv
├── bin # ipython, jupyterなどもここに保存される
│   ├── activate # 仮想環境の有効化のためのコマンド
│   ├── pip
│   ├── pip3
│   ├── python -> python3 # シンボリックリンク
│   └── python3 -> /usr/local/bin/python3 # シンボリックリンク
├── include
├── lib
│   └── python3.7 # 標準ライブラリはコピーされない
│       └── site-packages
└── pyvenv.cfg
```

---

### 標準的なPythonの環境構築｜仮想環境による構築

仮想環境を有効化する前後で`PATH`が書き換わることを確認する

```shell
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

$ which python3
/usr/local/bin/python3
```

```shell
$ source .venv/bin/activate
(.venv) $ echo $PATH
/path/to/project/.venv/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

(.venv) $ which python3
/path/to/project/.venv/bin/python3
```

---

### 標準的なPythonの環境構築｜仮想環境による構築

- 仮想環境への外部ライブラリのインストール

```shell
(.venv) $ pip install -q numpy
(.venv) $ pip list
Package    Version
---------- -------
numpy      1.18.5
pip        20.1.1
setuptools 41.2.0
```

- 仮想環境の無効化

```shell
(.venv) $ deactivate
```

---

### ツールを使ったPythonの環境構築｜概要

- 仮想環境は便利だけれど…
    - :slightly_frowning_face: 仮想環境の作成・使用に必要なステップが多い
    - :slightly_frowning_face: 仮想環境の出入り（有効・無効化）が面倒
    - :slightly_frowning_face: 外部ライブラリのインストールが手作業
- パッケージ管理ツール
    - :slightly_smiling_face: 仮想環境の作成・使用を簡単にする
    - :slightly_smiling_face: 外部ライブラリを設定ファイルで管理する
    - :slightly_smiling_face: その他、ツールごとにプラスαの機能を提供する

---

### ツールを使ったPythonの環境構築｜概要

- 代表的なツール
    - [Pipenv](https://pipenv-ja.readthedocs.io/ja/translate-ja)：PyPA（Python Packaging Authority）が開発しているツール。2017年頃登場。2020年6月現在最も知名度の高いツールで、日本語の情報も多い。データ解析・制御向け（後述）。
    - [Poetry](https://python-poetry.org/)：Sébastien Eustaceらコミュニティによって開発されているツール。2018年頃登場。パッケージ開発向け（後述）。
- [ベストプラクティス？](https://qiita.com/sk217/items/43c994640f4843a18dbe)
    - 2020年6月現在、様々なツールが乱立している状態のため、標準的な方法は存在しない。が、上記の2つなら大丈夫だろう。

---

### ツールを使ったPythonの環境構築｜概要

- ツールごとの違い
    - Pipenv：プロジェクトに必要な環境変数を`.env`ファイルに定義しておくと自動的に読み込んでくれる。例えば、ローカルに保存されたデータのディレクトリ名を環境変数で定義することで、ディレクトリ名をスクリプトに直書きせずに開発できる。
    - Poetry：Pythonパッケージの公開に必要な諸々のファイル（setup.py, MANIFEST.in）などを、設定ファイルの情報から自動生成してくれる。PyPI（Python Package Index）への登録もコマンド一つで完結する。

---

### ツールを使ったPythonの環境構築｜Pipenv

PipenvのインストールはHomebrewで行える。

```shell
$ brew install pipenv
```

Pipenvが仮想環境をプロジェクト配下に作成するよう設定。

```shell
$ # bashの場合
$ echo "export PIPENV_VENV_IN_PROJECT=true" >> ~/.bash_profile
$ source ~/.bash_profile

$ # zshの場合
$ echo "export PIPENV_VENV_IN_PROJECT=true" >> ~/.zprofile
$ source ~/.zprofile
```

---

### ツールを使ったPythonの環境構築｜Pipenv

- 設定ファイル
    - Pipfile：プロジェクトに必要なPythonのバージョンと外部ライブラリのバージョンの**条件**を記述する。例えば`numpy>=1.18`など。
    - Pipfile.lock：Pipenvによってインストールされた外部ライブラリの**実際の**バージョンの情報が保存される（自動生成される）。

[a-lab-nagoya/python-tutorials](https://github.com/a-lab-nagoya/python-tutorials)の設定を使って試してみよう。

```shell
$ git clone https://github.com/a-lab-nagoya/python-tutorials.git
$ cd python-tutorial
```

---

### ツールを使ったPythonの環境構築｜Pipenv

`pipenv install`で設定ファイル（Pipfile.lock）に保存された通りの情報で仮想環境を作成し、外部ライブラリをインストールする。

```shell
$ pipenv install --dev
```

`pipenv run <command>`で、仮想環境に出入りすることなく、仮想環境下のコマンドを実行することができる。例えば以下の通り。

```shell
$ pipenv run jupyter lab
```

---

### ツールを使ったPythonの環境構築｜Poetry

PoetryのインストールはHomebrewで行える。

```shell
$ brew install poetry
```

Poetryが仮想環境をプロジェクト配下に作成するよう設定。

```shell
$ # bashの場合
$ echo "export POETRY_VIRTUALENVS_IN_PROJECT=true" >> ~/.bash_profile
$ source ~/.bash_profile

$ # zshの場合
$ echo "export POETRY_VIRTUALENVS_IN_PROJECT=true" >> ~/.zprofile
$ source ~/.zprofile
```

---

### ツールを使ったPythonの環境構築｜Poetry

- 設定ファイル
    - pyproject.toml：プロジェクトに必要なPythonのバージョンと外部ライブラリのバージョンの**条件**を記述する。例えば`numpy>=1.18`など。
    - poetry.lock：Pipenvによってインストールされた外部ライブラリの**実際の**バージョンの情報が保存される（自動生成される）。

[a-lab-nagoya/python-tutorials](https://github.com/a-lab-nagoya/python-tutorials)の設定を使って試してみよう。

```shell
$ git clone https://github.com/a-lab-nagoya/python-tutorials.git
$ cd python-tutorial
```

---

### ツールを使ったPythonの環境構築｜Poetry

`poetry install`で設定ファイル（poetry.lock）に保存された通りの情報で仮想環境を作成し、外部ライブラリをインストールする。

```shell
$ poetry install
```

`poetry run <command>`で、仮想環境に出入りすることなく、仮想環境下のコマンドを実行することができる。例えば以下の通り。

```shell
$ poetry run jupyter lab
```

---

### 参考文献

- [2020 年の Python パッケージ管理ベストプラクティス - Qiita](https://qiita.com/sk217/items/43c994640f4843a18dbe)
- [Pipenv: 人間のためのPython開発ワークフロー](https://pipenv-ja.readthedocs.io/ja/translate-ja/)
- [Poetry - Python dependency management and packaging made easy.](https://python-poetry.org/)
- [Python Packaging Authority — PyPA documentation](https://www.pypa.io/en/latest/)
- [a-lab-nagoya/python-tutorials: Python tutorials based on chainer/tutorials (https://tutorials.chainer.org)](https://github.com/a-lab-nagoya/python-tutorials)
- [PATHを通すとは？ (Mac OS X) - Qiita](https://qiita.com/soarflat/items/09be6ab9cd91d366bf71)
