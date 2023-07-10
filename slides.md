---
marp: true
theme: gaia
paginate: true
backgroundImage: url('https://marp.app/assets/hero-background.jpg')
---

# Construction of Python environments on macOS

![bg right:30% 75%](https://openastronomy.org/pyastro/images/pyastro_logo_150px.png)

Akio Taniguchi / 2023-07-11

---

### Preparation

当日までに、以下の準備と理解をお願いします。

- 手元（ローカル）のマシンでのPython 3の用意
    - 最新のmacOSであればデフォルトで入っているはず
    - なければ[Homebrew](https://brew.sh)で導入→`brew install python`
- Bash（zsh）の最低限のコマンドの理解
    - `cd`, `mv`, `rm`, `mkdir`, `ls`あたりが使えればOKです
- Bash（zsh）の環境変数`PATH`の理解
    - いわゆる「[パスを通す](https://qiita.com/soarflat/items/09be6ab9cd91d366bf71)」とは何かが分かればOKです

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

### Pythonのディレクトリ構造（macOS Ventura）

```shell
$ which python3
/usr/bin/python3
```

```shell
$ which pip3
/usr/bin/pip3
```

```shell
$ python3 -m site
sys.path = [
    '/Users/<user name>',
    '/Library/Developer/CommandLineTools/.../3.9/lib/python39.zip',
    '/Library/Developer/CommandLineTools/.../3.9/lib/python3.9',
    '/Library/Developer/CommandLineTools/.../3.9/lib/python3.9/lib-dynload',
    '/Library/Developer/CommandLineTools/.../3.9/lib/python3.9/site-packages',
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

```shell
$ tree -L 3 .venv
.venv
├── bin # ipython, jupyterなどもここに保存される
│   ├── activate # 仮想環境の有効化のためのコマンド
│   ├── pip
│   ├── pip3
│   ├── python -> python3 # シンボリックリンク
│   └── python3 -> /Library/Developer/.../python3 # シンボリックリンク
├── include
├── lib
│   └── python3.9 # 標準ライブラリはコピーされない
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
numpy      1.25.1
pip        23.1.2
setuptools 58.0.4
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
    - [Pipenv](https://pipenv-ja.readthedocs.io/ja/translate-ja)：PyPA（Python Packaging Authority）が開発しているツール。2017年頃登場。データ解析・制御向け（後述）。
    - [Poetry](https://python-poetry.org/)：Sébastien Eustaceらコミュニティによって開発されているツール。2018年頃登場。パッケージ開発向け（後述）。
- ベストプラクティス？
    - 2023年現在、唯一の方法と言えるものはない。何をやりたいかで使い分けよう。迷ったら上記2つから選ぶのが良いだろう。（ただし、A研としては使用実績からPoetryを推奨）

---

### ツールを使ったPythonの環境構築｜概要

- ツールごとの違い
    - [Pipenv](https://pipenv-ja.readthedocs.io/ja/translate-ja)：プロジェクトに必要な環境変数を`.env`ファイルに定義しておくと自動的に読み込んでくれる。これにより、ユーザごとに異なる情報（例：ユーザ名）や共有すべきでない情報（例：パスワード・APIキー）など直書きせずに開発できる。
    - [Poetry](https://python-poetry.org/)：Pythonパッケージの公開に必要な諸々のファイル（`setup.py`, `MANIFEST.in`, ...）を、設定ファイル（`pyproject.toml`）の情報から自動生成してくれる。PyPI（Python Package Index）への登録もコマンド一つで完結する。

---

### Poetryを使ったPythonの環境構築｜ツールの導入

Poetryのインストールはコマンドラインで行う。

```shell
$ curl -sSL https://install.python-poetry.org | python3 -
$ brew install poetry # "This build of python ... symlinks"に遭遇したらこっちを試す
```

Poetryが仮想環境をプロジェクト配下に作成するように設定しておくと分かりやすい（デフォルトでは全く別の場所に作られる）。

```shell
$ poetry config virtualenvs.create true
```

---

### Poetryを使ったPythonの環境構築｜初期設定

プロジェクト（環境構築したいディレクトリ）を作成し、移動後`poetry init`で初期設定を行う。`pyproject.toml`が生成される。

```shell
$ mkdir /path/to/project
$ cd /path/to/project
$ poetry init
Package name [project]: <プロジェクト名>
Version [0.1.0]: <バージョン>
Description []: <プロジェクトの説明>
Author [<開発者名>, n to skip]:
License []: <ライセンスの種類>
Compatible Python versions [^3.11]: <プロジェクトがサポートするPythonのバージョン（後述）>
Would you like to define your main dependencies interactively? (yes/no) [yes] no
Would you like to define your development dependencies interactively? (yes/no) [yes] no
Do you confirm generation? (yes/no) [yes] yes
```

---

### Poetryを使ったPythonの環境構築｜初期設定

```shell
$ cat pyproject.toml
[tool.poetry]
name = "project"
version = "0.1.0"
description = ""
authors = ["<開発者名>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.11"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

---

### Poetryを使ったPythonの環境構築｜初期設定

`poetry add`で外部ライブラリのインストールを行う。仮想環境も同時に生成される。インストールが終わると`poetry.lock`が生成される。

```shell
$ poetry add numpy matplotlib
Creating virtualenv project in /path/to/project/.venv
Using version ^3.7.2 for matplotlib
Using version ^1.25.1 for numpy
...
Package operations: 11 installs, 0 updates, 0 removals
  • Installing numpy (1.25.1)
  • Installing matplotlib (3.7.2)
  ... # numpy, matplotlib以外の必要パッケージもインストールされる
Writing lock file
```

---

### Poetryを使ったPythonの環境構築｜コマンド実行

`poetry run`で仮想環境に「入った」状態でコマンドを実行ができる。

```shell
$ poetry run python /path/to/script.py # 仮想環境のPython 3で実行される
$ # コマンドの実行が終わると仮想環境の外に自動的に出る
```

`poetry shell`で仮想環境に入ったままの状態にもできる。

```shell
$ poetry shell
(project-py3.11) $ python # 仮想環境のPython 3が起動する
(project-py3.11) $ deactivate # venvと同様にdeactivateで抜ける
$
```

---

### Poetryを使ったPythonの環境構築｜設定ファイル

- **pyproject.toml：** プロジェクトに必要なPythonのバージョンと外部ライブラリのバージョンの**条件**を記述する。`poetry add`で自動記述されるが、条件を変えたい場合は自分で編集することになる。`poetry add numpy==1.20`のように追加時に指定することも可能。
- **poetry.lock：** Poetryによって仮想環境にインストールされた外部ライブラリの**実際の**バージョンの情報が保存される。`poetry update`で`pyproject.toml`に書かれた条件に従って仮想環境を都度更新できる。基本的にこのファイルはユーザが編集してはならない。

---

### Poetryを使ったPythonの環境構築｜バージョン指定

| 表記（例） | 条件 |
| :--- | :--- |
| `python = ">=3.8, <3.12"` | 3.8以上かつ3.12未満のPythonのみ許容（`,`はAND） |
| `numpy = "^1.20"` | 1.20以上、2.0未満のnumpyのみ許容（メジャーバージョンを上げてはいけない） |
| `pandas = "^1.5 \| ^2.0"` | 1.5以上かつ2.0未満、または2.0以上かつ3.0未満のみ許容（`\|`はOR） |

---

### ツールを使ったPythonの環境構築｜開発の流れ

- **環境作成・保存：** `poetry init`, `poetry add`で設定ファイル（`pyproject.toml`, `poetry.lock`）を一から作成。これらをGitHub等のクラウドで管理しておくことで、開発環境を「保存」する。
- **環境復元：** 設定ファイルを新しいマシンにコピーし、シングルコマンド（`poetry install`）でコピー元と同一の環境を「復元」する。
- **環境更新：** `pyproject.toml`を編集してPythonや外部ライブラリの条件を更新し、`poetry update`で実際にインストールされたものも更新することで、開発環境を「更新」する。
