python-my-project
=================

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor
incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis
nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore
eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt
in culpa qui officia deserunt mollit anim id est laborum.


Quick Links
----------

* [API Documents](...)
* [Issue Tracker](...)
* [Private Registry](...)


利用手順
----------

パッケージは GitLab Package Registry にアップロードされているため、ここからダウンロードします。

PIP の利用手順は次の通りです。

    $ pip install \
        --extra-index-url http://gitlab.example.com/api/v4/projects/1/packages/pypi/simple \
        --trusted-host gitlab.example.com \
        my-project

pip において http からのダウンロードを行うには、trusted-host に追加する必要があります。
PIP 設定ファイル中の `trusted-host` に URL を追加することで、`--trusted-host` の指定は不要になります。

    # pip の設定を書き込む
    # --site オプションを用いると、当該 venv に対してのみ設定を適用できる
    # 指定しない場合は、ユーザー単位の設定(--user)とみなされる
    # 参照: https://pip.pypa.io/en/stable/topics/configuration/
    $ pip config set --site global.trusted-host gitlab.example.com
    Writing to xxx/.venv/pip.conf

    $ pip install \
        --extra-index-url http://gitlab.example.com/api/v4/projects/1/packages/pypi/simple \
        my-project

Hatch の利用手順は次の通りです。
Hatch はバックエンドに PIP を利用します。
PIP は設定を環境変数経由で取得できるため、
dependencies に依存関係を定義することに加えて、
PIP_EXTRA_INDEX_URL、および PIP_TRUSTED_HOST に GitLab Package Registry を指定します。 

    $ cat .pyproject
       :
    dependencies = [
      "my-project",
    ]
        :
    [tool.hatch.envs.default.env-vars]
    PIP_EXTRA_INDEX_URL = "http://gitlab.example.com/api/v4/projects/1/packages/pypi/simple/ https://pypi.org/simple/"
    PIP_TRUSTED_HOST = "gitlab.example.com"
        :

参考:

* <https://docs.gitlab.com/ee/user/packages/pypi_repository/>
* <https://hatch.pypa.io/latest/config/dependency/>
* <https://hatch.pypa.io/latest/how-to/environment/package-indices/>
* <https://pip.pypa.io/en/stable/cli/pip/#trusted-host>
* <https://pip.pypa.io/en/stable/topics/configuration/>


開発手順
----------

プロジェクト構成は [pyproject.toml](./pyproject.toml) に全て記載されているので、参照ください。

### 動作環境の準備

プロジェクトは [Hatch](https://hatch.pypa.io/) を前提としています。

1. Python 3 (>3.9) をインストールします。
   また、シェルの rc ファイルを編集し、ユーザーインストールパス(`~/.local/bin`)にパスを通します。

2. pipx を利用可能にします。
   pipx はパッケージごとに独立した仮想環境(`~/.local/pipx/venvs/{PACKAGE_NAME}/`)にインストールする仕組みです。
   各パッケージのコマンドは通常のユーザーインストールパス(`~/.local/bin`)からシンボリックリンクされます。
   なお pip は必要に応じてインストールします。

        # Ubuntu の手順
        $ sudo apt install -y pipx python3-pip

        # RHEL の手順
        $ sudo dnf install -y pipx python3-pip

3. pipx で hatch をインストールします。
   また、black、mypy、ruff をインストールすると VSCode に認識され便利です。

        $ pipx install hatch
        $ pipx install black
        $ pipx install mypy
        $ pipx install ruff

参考:

* <https://hatch.pypa.io/latest/install/>

### 仮想環境の準備

デフォルトの venv 環境は、プロジェクトルートの `.venv` に展開されるように設定されています。

    # default 環境の生成
    $ hatch env create

デフォルト環境には次のものが含まれます。

* パッケージの依存関係
* デフォルト環境に定義された依存関係

Hatch の流儀に乗っ取り、デフォルト環境にはパッケージの実行、テストに必要なものしかはいっていません。


デフォルト以外に、次の環境が定義されています。

* docs: APIドキュメントビルド用環境
* lint: コードリント用環境

### 仮想環境の利用

仮想環境のシェルを開くには、`hatch shell` を使うのが簡単です。

    # venv 環境でシェルを開く
    $ hatch shell

初回の `hatch shell` 実行時、自動で `hatch env create` されます。

異なる環境のシェルを開くには、`hatch env run` コマンドを用います。

    # lint 環境で zsh を起動
    $ hatch env run -e lint zsh

なお VSCode で Python 拡張を利用すると、プロジェクトルートに `.venv` がある場合、
ターミナルを開いた際、自動で venv が activate されるため、`hatch shell` は不要です。
ただしこの場合、あらかじめ `hatch env create` しておく必要があります。

また、VSCode で Jupyter 拡張を利用すると、仮想環境上で Jupyter を実行できるため、
デバッグに便利です。"Create: New Jupyter Notebook" で Jupyter Notebook を開き、
環境に既存の .venv を選択してください。

参考:

* <https://hatch.pypa.io/latest/environment/>
* <https://code.visualstudio.com/docs/languages/python>

### テストの実行

テストは pytest を利用します。
pytest-cov を利用したカバレッジ計測も利用できます。

    # テストを実行
    $ hatch run test

    # カバレッジレポートあり
    $ hatch run test-cov

テスト実行には、次の環境変数を設定してください。

* `...` ...

`.env` ファイルによる設定にも対応しています。

`$PROJECT_ROOT`/.env`:

    ...=...

.env ファイルについては、[python-dotenv](https://github.com/theskumar/python-dotenv)を参照ください。

VSCode で Python Extension を使うと、[テストパネルから実行](https://code.visualstudio.com/docs/python/testing)できます。

### コードの整形・解析

ruff によるコード解析、black によるコードフォーマット、mypy による型チェックが利用可能です。

    # 全 lint を実行
    $ hatch run lint:all

    # コードを自動で整形
    $ hatch run lint:format

### ドキュメントの生成

pdoc による API ドキュメント生成が利用可能です。

    $ hatch run docs:build
    # -> docs/_build に生成される

### パッケージの生成

hatch によるパッケージビルド手順を示します。

    $ hatch build

参考:

* <https://hatch.pypa.io/latest/build/>

### リリース

リリース前に、次のことを確認してください。

* コードフォーマット警告が無いこと
* テストが全て通ること
* ドキュメントがコードの変更に追従していること
* パッケージがビルドできること

リリース時には、次の作業を行います。

1. リリースのためのコミットを行う
  * 変更履歴を <HISTORY.md> に追記
  * `__version__.py` の `__version__` 変数を更新
2. コミットにタグを打つ (`vX.X.X` 形式)
3. コミット、タグを push する

タグの操作方法を示します。

    # 例: v0.0.5dev の場合
    $ git tag "v0.0.5dev"

    # 例: v1.0.0 の場合
    $ git tag "v1.0.0"

    # tag を push する
    $ git push origin --tags

    # tag を消す場合
    $ git push --delete origin TAG_NAME

### 参考: パッケージ公開手順

パッケージは GitLab Package Registry にアップロードします。
通常、tag を付与すると自動でアップロードされるため、手動でのアップロードは不要です。

以下では、参考情報として、手動でアップロードする手順を示します。

事前に、[パッケージをビルド](#パッケージの生成)しておきます。

hatch を用いてアップロードを行います。

    # 本来は次の手順でアップロードできるはずだがバグっているので使わない
    # $ hatch publish -r private-gitlab-1 -a ${GITLAB_PERSONAL_ACCESS_TOKEN}

    # かわりに、手動で URL を指定する。
    $ hatch publish -r http://gitlab.example.com/api/v4/projects/1/packages/pypi -u oauth2 -a ${GITLAB_PERSONAL_ACCESS_TOKEN}

hatch のかわりに twine を用いることもできます。
この場合、`.pypirc` が利用できます。

    # .pypirc を設定する
    $ cat ~/.pypirc
    [distutils]
    index-servers =
        private-gitlab-1

    [private-gitlab-1]
    repository = http://gitlab.example.com/api/v4/projects/1/packages/pypi
    username = oauth2
    password = <GITLAB_PERSONAL_ACCESS_TOKEN>

    # twine でアップロードを行う
    $ twine upload -r private-gitlab-1 dist/*

参考:

* <https://hatch.pypa.io/latest/publish/>
* <https://docs.gitlab.com/ee/user/packages/pypi_repository/>
* <https://packaging.python.org/ja/latest/specifications/pypirc/>
