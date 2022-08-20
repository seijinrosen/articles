---
title: 【Python】argparse で `--help` や `--version` のようなコマンドを自作する
tags: Python Python3 argparse cli
author: seijinrosen
slide: false
---
[argparse](https://docs.python.org/ja/3/library/argparse.html) を使って、一般的な CLI における `--help` や `--version` のような動きをするコマンドを実装してみました。

具体的には以下の要件です。

- 他の引数やオプションよりも優先して実行される
- 実行された後、即時終了する

## 実行環境

`Python 3.9.7` で正常に実行されることを確認しました。ある程度新しいバージョンであれば動くと思います。

## サンプルスクリプト

`-S` もしくは `--system` をオプションとして与えられたとき、`print(sys.version)` を実行して処理を終了するシンプルなスクリプトです。

```python:sample.py
import sys
from argparse import SUPPRESS, Action, ArgumentParser


def show_system_version():
    print(sys.version)


class SampleAction(Action):
    def __init__(self, option_strings, dest=SUPPRESS, default=SUPPRESS, help=None):
        super().__init__(
            option_strings=option_strings,
            dest=dest,
            default=default,
            nargs=0,
            help=help,
        )

    def __call__(self, parser, namespace, values, option_string=None):
        show_system_version()
        parser.exit()


def main():
    parser = ArgumentParser()
    parser.add_argument(
        "-S", "--system", action=SampleAction, help="show system version and exit"
    )
    parser.add_argument("--mushi", help="無視されるべきオプション")
    args = parser.parse_args()
    print(args.mushi)


if __name__ == "__main__":
    main()
```

`show_system_version` 関数の中身やヘルプメッセージを変更するなどしてカスタマイズできます。

ポイントは以下です。

- `Action` クラスを継承したクラスを作成する
- `__call__` 内で処理を実行し、 `parser.exit()` する
- `add_argument` の `action` にそのクラスを指定する

実装の詳細は、[argparse モジュールの _HelpAction クラスと _VersionAction クラス](https://github.com/python/cpython/blob/9cd8fb8d6356c17dafa1be727cab3d438f6df53f/Lib/argparse.py#L1079-L1121) を参考にしました。

## サンプルスクリプトの実行結果

```sh
# ヘルプテキストが正常に表示されることを確認
$ python3 sample.py -h

usage: sample.py [-h] [-S] [--mushi MUSHI]

optional arguments:
  -h, --help     show this help message and exit
  -S, --system   show system version and exit
  --mushi MUSHI  無視されるべきオプション
```

```sh
# print(sys.version) が正常に実行されることを確認
$ python3 sample.py -S

3.9.7 (default, Sep  3 2021, 12:37:55)
[Clang 12.0.5 (clang-1205.0.22.9)]
```

```sh
# -S をつけないと --mushi 引数のテキストは表示される
$ python3 sample.py --mushi テキスト

テキスト
```

```sh
# -S をつけると --mushi は無視される
$ python3 sample.py --mushi テキスト -S

3.9.7 (default, Sep  3 2021, 12:37:55)
[Clang 12.0.5 (clang-1205.0.22.9)]
```

```sh
# 順番を変更しても上記と同じ結果
$ python3 sample.py -S --mushi テキスト

3.9.7 (default, Sep  3 2021, 12:37:55)
[Clang 12.0.5 (clang-1205.0.22.9)]
```

## 補足

サンプルスクリプト内で `sys.version` を表示していますが、実際に**パッケージやライブラリのバージョン**を表示する `--version` コマンドを実装する場合は、`add_argument('--version', action='version', version='%(prog)s 2.0')` のような専用アクションが用意されていますので、そちらを使用すべきです。

## 参考リンク

- [argparse --- action — Python 3.9.4 ドキュメント](https://docs.python.org/ja/3/library/argparse.html#action)
- [argparse --- Action クラス — Python 3.9.4 ドキュメント](https://docs.python.org/ja/3/library/argparse.html#argparse.Action)
- [cpython/argparse.py at 9cd8fb8d6356c17dafa1be727cab3d438f6df53f · python/cpython · GitHub](https://github.com/python/cpython/blob/9cd8fb8d6356c17dafa1be727cab3d438f6df53f/Lib/argparse.py#L1079-L1121)

## 終わりに

記事内に誤りがある場合や「もっとうまいやり方があるよ」という場合は、コメント頂けますと幸いです。

また、Qiita 初投稿ですので、記事の書き方の作法的な部分でのご指摘もお待ちしております。
