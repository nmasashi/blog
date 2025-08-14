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

### None

None は、C 言語や Java でいう`null`に相当するもの。条件式で使用する場合は偽となり、比較する場合は`==`などを使用せずに`is`や`is not`を使用する。

```py
if n is None:
    処理
```

### ブール演算

or, and, not の 3 種類。

- or と and は**bool 型の変数(`True` or `False`)が返るわけではなく**、真または偽に判定されたオブジェクトが返る。（※直接、`True` と `False`を比較していた場合は`True` と `False`のどちらかが返る）
- not は`True` か `False` が返る。

```py
x = ['book']
y = []
z = 1
print(x or y)   # ['book']
print(x and y)  # []
print(x or z)   # ['book']
print(x and z)  # 1
print(z and x)  # ['book']
print(not x)    # False
```

**感想** わかりにく！（笑）

- `x or y`は`x`が真なら`y`は評価されない
- `x and y`は`x`が偽なら`y`は評価されない

### 数値どうしの演算

```py
print(1 + 2)    # 整数どおしの計算は整数
print(1 + 2.0)  # 小数が混じると結果は小数
print(7 / 2)    # 整数どおしでも小数になる
print(7 // 2)   # 小数点以下切り捨て
print(7 ** 2)   # 7の2乗
```

出力

```sh
3
3.0
3.5
3
49
```

小数を使用した結果比較は注意が必要（[詳細](https://docs.python.org/ja/3/tutorial/floatingpoint.html)）。

```py
print(0.1 + 0.1 + 0.1 == 0.3)
print(round(0.1 + 0.1 + 0.1, 1) == round(0.3, 1))   # roundを使用して小数点以下1桁で丸めている
```

出力

```sh
False
True
```

`round`や math モジュールの`math.isclose()`などで対応可能

- python は整数の精度に制限ないため、メモリの許す限り大きな値を扱える
- 無限大(`float(inf)`)は float 型
- 数値として扱えない値である NaN (not a number)も float 型(`float('nan')`)

### str 型

ダブルクオートかシングルクオートでくくると文字列を定義できる。3 つのクオートでくくると改行を含めた文字列にできる。
また、長い文字列を定義するときなどは`()`でくくることで分けて定義できる。

```py
notebook = """
note
book
"""

url = ('htts://hogehoge.com'
       '/fugafuga/hoge/fuga'
       '/2025/08/21')

print(notebook)
print(url)
```

出力

```sh

note
book

htts://hogehoge.com/fugafuga/hoge/fuga/2025/08/21
```

### 文字列の演算

`+`で結合できる。`*`を使用すると繰り返しになる。

```py
book = 'book'

print('note' + book)
print(book * 2)
```

出力

```sh
notebook
bookbook
```

### 条件式での文字列の利用

```py
print(bool(''))     # False
print(bool('book')) # True
print('oo' in 'book')   # True
print('oo' in 'book')   # True
print('oo' not in 'book')   # False
```

### f-string

```py
title = 'book'
print(f'This is a {title}')
```

出力

```sh
This is a book
```

`{}`でくくった式を利用して変数を確認することもできる。デバックに便利

```py
title = 'book'
print(f'{title=}')
print(f'{title.upper()=}')
```

出力

```sh
title='book'
title.upper()='BOOK'
```

### format()

基本的な使用方法。

```sh
print('python {} {}.'.format('practice', 'book'))
print('python {1} {0}.'.format('practice', 'book'))
print('python {p} {b}.'.format(p='practice', b='book'))
```

出力

```sh
python practice book.
python book practice.
python practice book.
```

アンパック（詳細は 5 章）を利用して辞書を渡すと、その辞書からキーワードをきにーして取得できた値に置換できる

```py
d = {'x': 'note', 'y': 'notebook', 'z': 'sketchbook'}
books = '{x} {z}'
print(books.format(**d))
```

出力

```sh
note sketchbook
```

### str 型とよく似た bytes 型

bytes 型と str 型は相互に変換が可能

```py
book = 'Python実践入門'
encoded = book.encode('utf-8')
print(encoded)
print(encoded.decode('utf-8'))
```

出力

```sh
b'Python\xe5\xae\x9f\xe8\xb7\xb5\xe5\x85\xa5\xe9\x96\x80'
Python実践入門
```

### list 型

要素の型はそろってなくて OK。基本的な操作。

```py
items = ['note', 'notebook', 'sketchbook']
print(items)

items.append('paperbook')
print(f'追加：{items}')

items = ['book'] + items
print(f'結合：{items}')

print(f'pop：{items.pop(0)}')
print(f'pop後：{items}')

del items[1]
print(f'削除：{items}')
```

出力

```sh
['note', 'notebook', 'sketchbook']
追加：['note', 'notebook', 'sketchbook', 'paperbook']
結合：['book', 'note', 'notebook', 'sketchbook', 'paperbook']
pop：book
pop後：['note', 'notebook', 'sketchbook', 'paperbook']
削除：['note', 'sketchbook', 'paperbook']
```

様々な方法での要素へのアクセス

```py
items = ['note', 'notebook', 'sketchbook']
print(items)

print(f'最後尾：{items[-1]}')
print(f'最初の二つ：{items[0:2]}')
print(f'最初の二つ：{items[:2]}')
print(f'先頭を飛ばして最後まで：{items[1:]}')

items[0:2] = [1, 2, 3]
print(f'一部置き換え：{items}')
```

出力

```sh
['note', 'notebook', 'sketchbook']
最後尾：sketchbook
最初の二つ：['note', 'notebook']
最初の二つ：['note', 'notebook']
先頭を飛ばして最後まで：['notebook', 'sketchbook']
一部置き換え：[1, 2, 3, 'sketchbook']
```

### tuple 型

不変な配列を扱う型。定義後は変更不可能。

作成方法

```py
items = ('note', 'notebook', 'sketchbook')
print(items)
```

出力

```sh
('note', 'notebook', 'sketchbook')
```

要素へのアクセス方法は list 型と同じ。不変なので、変更しようとするとエラーになる。

### 条件式で使える配列の性質

- 要素が 1 つもない空の状態だと偽になる
- in 演算子を使用すると任意の要素があるか確認できる

```py
print('note' in ['note', 'notebook', 'sketchbook']) # True
```

### dict 型

いわゆる連想配列やマップや辞書型配列のようなもの(key-value ストア)

- key に使用できるのは文字列、数値、タプルなどの不変オブジェクト（= list 型は不可能）
- 空の場合は偽になる

基本的な追加削除の操作

```py
items = {'note': 1, 'notebook': 2, 'sketchbook': 3}
print(items)

items['book'] = 4
print(f'要素追加：{items}')

print(f'要素削除：{items.pop('notebook')}')
print(f'要素削除後：{items}')

del items['sketchbook']
print(f'要素削除後(del)：{items}')
```

出力

```sh
{'note': 1, 'notebook': 2, 'sketchbook': 3}
要素追加：{'note': 1, 'notebook': 2, 'sketchbook': 3, 'book': 4}
要素削除：2
要素削除後：{'note': 1, 'sketchbook': 3, 'book': 4}
要素削除後(del)：{'note': 1, 'book': 4}
```

要素の削除前に値が欲しい場合は`pop`を利用する。`del`を使用すると値の取得はできない。

要素へのアクセスは基本的に`get`を使用する。`items['note']`のような書き方でも値は取得できるが、キーがない場合は例外が発生する。

```py
items = {'note': 1, 'notebook': 2, 'sketchbook': 3}

print(items['note'])
print(items['book'])
```

出力

```sh
1
Traceback (most recent call last):
  File "sample.py", line 17, in <module>
    print(items['book'])
          ~~~~~^^^^^^^^
KeyError: 'book'
```

`get`を使用するとキーがない場合はデフォルトで`None`が返る。デフォルトは`get`の引数で変更可能

```py
items = {'note': 1, 'notebook': 2, 'sketchbook': 3}

print(items.get('note'))    # 1
print(items.get('book'))    # None
print(items.get('book', 0)) # 0
```

for 文での挙動

```py
items = {'note': 1, 'notebook': 2, 'sketchbook': 3}

# keyだけ
for key in items:
    print(key)

# valueだけ
for value in items.values():
    print(value)

# key, valueどちらも
for key, value in items.items():
    print(key, value)
```

出力

```sh
note
notebook
sketchbook
1
2
3
note 1
notebook 2
sketchbook 3
```

in 演算子を使用すると key または value に値が存在するか確認可能

```py
items = {'note': 1, 'notebook': 2, 'sketchbook': 3}

# keyの存在確認
print('note' in items)  # True
print('book' in items)  # False
# valueの存在確認
print(1 in items.values())  # True
```

### set 型

- 可変オブジェクト
- 一意の要素の集合を扱う（重複は許さない）
- 要素の順番を保持しないのでインデックスでのアクセス(`items[0]`みたいなの)は不可能

```py
items = {'note', 'notebook', 'sketchbook'}
print(items)

items.add('book')
print(f'追加: {items}')

items.remove('book')
print(f'削除: {items}')

# 順序を持たないため、popはどこから削除されるのか不定
print(f'pop: {items.pop()}')
print(f'削除(pop): {items}')
```

出力

```sh
{'sketchbook', 'note', 'notebook'}
追加: {'sketchbook', 'note', 'notebook', 'book'}
削除: {'sketchbook', 'note', 'notebook'}
pop: sketchbook
削除(pop): {'note', 'notebook'}
```

### frozenset 型

- set 型を不変にした型
- set 型と同様に、重複を許さず順序の保証もない

作成方法

```py
items = frozenset(['note', 'notebook'])
```

### 集合の演算

```py
set_a = {'note', 'notebook', 'sketchbook'}
set_b = {'book', 'rulebook', 'sketchbook'}

print(f'和集合: {set_a | set_b}')   # set_a.union(set_b)も同じ
print(f'差集合: {set_a - set_b}')   # set_a.difference(set_b)も同じ
print(f'積集合: {set_a & set_b}')   # set_a.intersection(set_b)も同じ
print(f'対称差: {set_a ^ set_b}')   # 和集合から積集合を引いたもの。set_a.symmetric_difference(set_b)も同じ
```

出力

```sh
和集合: {'sketchbook', 'notebook', 'rulebook', 'note', 'book'}
差集合: {'note', 'notebook'}
積集合: {'sketchbook'}
対称差: {'note', 'book', 'notebook', 'rulebook'}
```

部分集合の判定

```py
set_a = {'note', 'notebook', 'sketchbook'}
print({'note', 'notebook'} <= set_a)    # True
```

set 型と frozset 型の for 文も list 型と同じような書き方(`for item in items`)が可能だが、要素の順番が不定なのは注意が必要

### 内包表記

例えば、`['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']`というリストを作成したい場合、内包表記を使用しない場合は下記のようなものになる

```py
numbers = []
for i in range(10):
    numbers.append(str(i))
```

内包表記を利用すると下記のようになる。

```py
numbers = [str(v) for v in range(10)]
```

内包表記は`[リストの要素 for 変数 in イテラブルなオブジェクト]`という構文。
また、内包表記で使用した変数のスコープは内包表記内で閉じている。
つまり`[str(v) for v in range(10)]`の`v`はこの内包表記内がスコープ。

このような入れ子構造も可能

```py
tuples = [(x, y) for x in [1, 2, 3] for y in [4, 5, 6]]
# [(1, 4), (1, 5), (1, 6), (2, 4), (2, 5), (2, 6), (3, 4), (3, 5), (3, 6)]
```

入れ子にした分、可読性が下がるのでそこは普通の for 文を使用するか要判断

if 文も可能

```py
even = [x for x in range(10) if x % 2 == 0]
# [0, 2, 4, 6, 8]
```

- 内包表記で集合を作成することも可能(例：`{v for v in range(10)}`)
- 内包表記で辞書を作成することも可能(例：`{str(v): v for v in range(10)}`)
  - `{'0': 0, '1': 1, ...}`

## 第 5 章 関数

### 関数の定義と実行

基本的な構文

```py
def 関数名(引数1, 引数2, ...)
    処理
    return 戻り値
```

`return`はなくても OK。ない場合は`None`が返る

引数のデフォルト値を指定できる。ただし、不変オブジェクトを指定すること（可変オブジェクトも指定可能だがバグのもとになる）。

```py
def print_page(content='no content'):
    print(content)
```

### 関数はオブジェクト

関数もオブジェクトなので変数に代入することができる

```py
def print_page(content='no content'):
    print(content)

# 変数fに関数を代入
f = print_page
f()
```

出力

```sh
no content
```

### 関数のさまざまな引数

何も指定がなければ順番通りにあてはめられるが、引数名で指定も可能。
位置と引数名の組み合わせも可能だが、位置でのしてより後ろに引数名での指定を持ってくる必要がある。

```py
def minus(a, b):
    return a - b


# 位置で指定
print(minus(1, 2))

# 引数名で指定
print(minus(b=1, a=2))

# 位置と引数名の組み合わせも可能
print(minus(1, b=2))
# print(minus(a=1, 2))←これはダメ
```

引数名に`*`を付けることで可変長の引数を定義可能。慣例として`*args`とされることが多い。

- 関数は`*args`はタプルとして受け取る
- `*args`が空の場合は空のタプルになる
- 位置引数より後にしておいた方が良い（良くない例：`def print_pages(*args, content):`）

```py
def print_pages(content, *args):
    print(content)
    for more in args:
        print(f'more: {more}')


print_pages('content name', 'contentA', 'contentB', 'contentC')
```

出力

```sh
content name
more: contentA
more: contentB
more: contentC
```

引数名に`**`を付けることで可変長のキーワード引数を定義可能。慣例として`**kwargs`とされることが多い。

```py
def print_pages(*args, **kwargs):
    for more in args:
        print(f'more: {more}')
    for key, value in kwargs.items():
        print(f'{key}: {value}')


print_pages('contentA', 'contentB', 'contentC',
            published=2019, author='rei suyama')
```

出力

```sh
more: contentA
more: contentB
more: contentC
published: 2019
author: rei suyama
```

※可変長の引数は便利だけど可読性に問題が出る場面もあるので、注意して使用すること

キーワードのみ引数で呼び出し時に引数名の指定を強制することができる。ユーザーに引数の意味を意識させたり、可読性の向上に効果がある。

下記のように、キーワードのみ引数にしたい引数の前に`*`を指定する

```py
def increment(page_num, last, *, ignore_error=False):
    処理

increment(1, 2, ignore_error=True)

# これはエラー
# increment(1, 2, True)
```

一方で、位置のみ引数にしたい場合はその引数の後に`/`を指定することで実現できる

```py
def add(x, y, /, z):
    処理

add(1, 2, z=3)

# これはエラー
# add(x=1, y=2, z=3)
```

### 引数リストのアンパック

関数呼び出し時に`*`演算子をリストに使用すると引数を展開して渡してくれる(JS でいうところのスプレッド構文的な感じ？)。
辞書型を値を展開して渡した場合は`**`演算子を使用する。

```py
def print_page(one, two, three):
    print(one)
    print(two)
    print(three)


items = ['mycontent', 'contentA', 'contentB']
print_page(*items)
```

出力

```sh
mycontent
contentA
contentB
```

### lambda 式

基本的な構文は下記のもので、一行で記述する必要がある。

```py
lambda 引数1, 引数2, ...: 戻り値になる式
```

```py
def increment(num): return num + 1

print(increment(2)) # 3
```

`filter`や`sorted`などで使用されることが多い

```py
nums = ['one', 'two', 'three']

filtered = filter(lambda x: len(x) == 3, nums)
print(list(filtered))
```

出力

```sh
['one', 'two']
```

### 型情報の付与

- 関数にはアノテーションを用いて型ヒントを追加できる
- 実行時に型チェックが実施されるわけではない（= 違反してもエラーにはならない）
- 保守性の向上に効果がある
- 静的解析ツール mypy を活用したチェックで一致していない呼び出し個所のチェックが可能

基本的な構文

```py
def 関数名(arg: arg1の型, arg2: arg2の型, ...) -> 戻り値の型:
    処理
    return 戻り値
```

使用例

```py
def decrement(page_num: int) -> int:
    prev_page: int
    prev_page = page_num - 1
    return prev_page
```

## 第 6 章 クラスとインスタンス
