---
layout: post
title: "Python 実践入門"
date: 2025-08-05
categories: reading python
---

![image]({{site.baseurl}}/images/reading/python/bookcover.jpg)

この本読んだのでメモ

## 第 1 章 Python はどのような言語か

- 動的型付き言語だが、型がないわけではない

→ 整数と文字列の足し算と化するとエラーになる

- 処理ブロックはインデントで決まり、スペース 4 つのインデントが一般的
- 標準ライブライが豊富
  - json: json のエンコード・でコード
  - csv: csv の読み書き
  - zipfile: zipfile の処理
- python は OSS で誰でも開発に参加可能
  - [github リポジトリ](https://github.com/python/cpython)
- python のコーディング規約
  - [PEP8 - Style Guide for Python Code](https://peps.python.org/pep-0008/)が一般的([日本語版](https://pep8-ja.readthedocs.io/ja/latest/))
- 実際に開発する際は自動成型ツールなどを使用する
  - [Flake8](https://flake8.pycqa.org/en/latest/)
  - [pycodestyle](https://pycodestyle.pycqa.org/en/latest/)
  - [Black](https://black.readthedocs.io/en/stable/)

## 第 2 章 Python の実行

対話モードでの実行

- `type(s)`でデータ型の確認
- `dir(s)`でそのオブジェクトが持つ属性（変数やメソッド）が確認できる
  - `__`が前後についているものは Python が暗黙的に利用するみたい（よくわからん）

```py
Python 3.12.3 (main, Jun 18 2025, 17:59:45) [GCC 13.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> s = 'Hello world'
>>> type(s)
<class 'str'>
>>> dir(s)
['__add__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__getstate__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'capitalize', 'casefold', 'center', 'count', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'format_map', 'index', 'isalnum', 'isalpha', 'isascii', 'isdecimal', 'isdigit', 'isidentifier', 'islower', 'isnumeric', 'isprintable', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'removeprefix', 'removesuffix', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']
>>> s.upper()
'HELLO WORLD'
```

よくわからない関数がある場合は`help`を利用する。

```py
>>> help(s.zfill)
```

こんな感じで内容を説明してくれる。

```txt
Help on built-in function zfill:

zfill(width, /) method of builtins.str instance
    Pad a numeric string with zeros on the left, to fill a field of the given width.

    The string is never truncated.
```

この`help`は Docstring から自動生成される

```py
>>> def increment(n):
...     """nに1足すやで
...     :param n:数値
...     :return: nに1足したやつ
...     """
...     return n + 1
...
>>> increment(3)
4
>>> help(increment)
```

ヘルプページ

```text
Help on function increment in module __main__:

increment(n)
    nに1足すやで
    :param n:数値
    :return: nに1足したやつ
```

- モジュール：python コード記述したファイル
- パッケージ：複数のモジュールを集めたもの

## 第 3 章 制御フロー

### pass 文

何もしないブロックを記述するときは pass 文を活用する。
下記のプログラムは`pass`がないとエラーになる。

```py
class PracticeBookError(Exception):
    pass
```

もしくは、docstring を使用する。

```py
class PracticeBookError(Exception):
    """モジュール独自の例外規定クラス"""

```

### 変数の利用

カンマ区切りで一度に代入可能

```py
x, y, z = 1, 2, 3
```

### コメント

```py
# この行はコメント
def comment(): # ここはコメント
    pass
```

**コメントと Docstring の違い**

docstring はモジュールの冒頭と関数、クラス、メソッドの定義で使用できる機能。

### if 文

基本形はこんな感じ

```py
if 条件式1:
    処理1
elif 条件式2:
    処理2
else:
    処理3
```

偽になる値

- None
- False
- 整数型のゼロ。0 や 0.0、0j（数値に j または J を付けると複素数になる）
- 文字列、リスト、辞書、集合などのコンテナオブジェクトの空オブジェクト
- メソッド`__bool__()`が False を返すオブジェクト
- メソッド`__bool__()`を定義していおらず、メソッド`__lent__()`が 0 を返すオブジェクト

上記に該当しないものは全て真

コンテナオブジェクトが空のことを利用するとシンプルな条件文が書ける

```py
def first_item(items):
    if len(items) > 0: #要素が空か判定
        return items[0]
    else:
        return None

def first_item(items):
    if items: #からのコンテナオブジェクトは偽になる
        return items[0]
    else:
        return None
```

**感想** シンプルだけど分かりにくいような気もする。python 慣れしてたら普通なのかな？

こんな感じで比較を連鎖させることも可能

```py
if x < y < z: # x < y and y < z と同等
    ...
```

### if 文でよく使うオブジェクトの比較

```py
items = ['book', 'note']
'book' in items # ← True
```

### for 文

**基本形**

```py
for 変数 in イテラブルなオブジェクト:
    繰り返したい処理
```

例

```py
for i in range(3):
    print(f'{i}番目の処理')
```

出力

```sh
0番目の処理
1番目の処理
2番目の処理
```

**カウントしたい場合**

```py
for カウント, 変数 in enumerate(イテラブルなオブジェクト):
    繰り返したい処理
```

例

```py
for count, char in enumerate('word'):
    print(f'{count}番目の文字は{char}')
```

出力

```sh
0番目の文字はw
1番目の文字はo
2番目の文字はr
3番目の文字はd
```

### for 文の else 節の挙動

```py
# breakの処理が実行されない場合、else句が実行される
print('パターン1: elseが実行される')
for n in [2, 4, 6, 8]:
    if n % 2 == 1:
        break
else:
    print('奇数を含めてください')

print('パターン2: elseが実行されない')
for n in [2, 4, 7, 8]:
    if n % 2 == 1:
        break
else:
    print('奇数を含めてください')
```

出力

```sh
パターン1
奇数を含めてください
パターン2
```

continue が動いても else には特に影響なし

```py
for n in [2, 3, 4, 6, 8]:
    if n == 3:
        continue
    elif n % 2 == 1:
        break
else:
    print('奇数を含めてください')
```

出力

```sh
奇数を含めてください
```

### for 文での変数スコープ

繰り返し用に使用した変数 `i` などは for 文を抜けても生きているので注意（普通の変数扱いになっている）

```py
for i in range(3):
    print(f'{i}番目の処理')

print(i)
```

出力

```sh
0番目の処理
1番目の処理
2番目の処理
2
```

### while 文

while 文も for 文と同じ感じ。else 句も使用可能(break されなければ実行される)

### try 文

```py
try:
    例外が発生する可能性がある処理
except 捕捉したい例外クラス:
    処理A
else:
    例外が発生しなかったときのみ実行される処理（exceptがある場合に記述可能）
finally:
    例外の発生有無にかかわらず実行したい処理
```

except 句は複数設定可能かつ一つの except 節で複数の例外を列挙可能(`except (Error1, Error2)`)。
また、複数の except 節を設定した場合は最初にマッチした except 節が実行される。

もし、例外が捕捉されなかった場合は（finally 節があれば、finally 節の処理後に）、その処理の呼び出し元に例外が送出される。

try 節には except 節で捕捉したい処理を記述し、else 節で例外監視の対象外の処理を記述することで、どの処理が例外発生の可能性がある処理なのかわかりやすくすることができる。

### raise 文

except 節で例外を捕捉したけど呼び出し元にも送出したい場合に利用できる。

下記の例だと、except 節で例外を捕捉すると`function`を呼び出したもとにも例外が送出される

```py
def function(param)
    try:
        例外が発生する可能性があるクラス
    except 捕捉したい例外クラス:
        処理A
        raise
```

### with 文

定義済みのクリーンナップ処理を実行してくれる。

```py
with open('some.txt', 'w') as f:
    f.write('some text')
f.closed # ← trueになる。withを使用しない場合、f.close()を実行しておく必要がある
```

with 文に対応したオブジェクトはコンテキストマネージャーと呼ばれる（9 章で詳しく説明）

## 4 章 データ構造
