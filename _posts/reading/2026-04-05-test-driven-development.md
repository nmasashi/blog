---
layout: post
title: "テスト駆動開発"
date: 2026-04-05
categories: reading tdd
---

![image]({{site.baseurl}}/images/reading/test_driven_dev/bookcover.jpg)

いい加減に読もうかと

ウォード・カニンガム(Ward Cunningham)さん：ソフトウェア開発の分野ですごい人

- 世界初のwiki
- 技術的負債
- CRCカード（Class-Responsibility-Collaboration cards）
- アジャイルマニフェストの著名者の一人

WyCash: ポートフォリオ管理システム

## 第Ⅰ部 多国通貨

### 仮実装

1. テストを書く
   - この時はコンパイルすら通らなくてOK
1. コンパイラを通す
   - とりあえずコンパイルは通るようにクラスや必要な関数を作る
1. テストを走らせ、失敗を確認する。
1. テストが通す
   - どんな方法でもいいから通す
1. 重複を排除する

みたいな感じで小さなステップを踏んでプロダクトコードを変更していく

### 原則をあえて破るとき

テストを通すためなら一時的にコピーアンドペーストでコードを複製するのもOK。もちろん後ほど重複を削除する前提。

### テスト不足にきづいたら

コピーして作ったクラスはその親クラスを定義して継承するような変更を加えて、さらにそのクラス用のテストを追加する

### 不要になったら消す

単体テストのコードも重複している内容や冗長なものがあれば削除する。プロダクトコードもそうだがテストコードも同様に削除対象になりうる。

## 第Ⅱ部 xUnit

jUnitを作るような話。jUnitを作るにもテスト駆動の考え方をつかってつるといいよ～てきなこと。
ちょっと難しくて頭にはいらなかった。jUnitつくることはないと思うし、まぁいいかな。。。

## 第Ⅲ部 テスト駆動開発のパターン

### 三角測量

プロダクトコードの仮実装をした際に、正しい一般化をする目的で使用する

```java
@Test
public void testSum(){
   assertEquals(4, plus(3, 1)); // これはpassする
   assertEquals(7, plus(3, 4)); // これはNGになる。ただし、本実装して正しく一般化すればpassする
}

// 一般化されていない仮実装
private int plus(int augend, int addend){
   return 4;
}
```

### Pluggable Object

`if`で振る舞いを切り替える代わりに、振る舞いごとにオブジェクトを作って差し替える（プラグみたいに挿し替える）パターン。
同じ条件の`if`があちこちのメソッドに出てきたら使いどころ。

例：図形エディタの選択ツール

- 図形の上でクリック → ドラッグで図形を動かす
- 何もないところでクリック → ドラッグで範囲選択

素直に書くと`mouseDown`・`mouseMove`・`mouseUp`それぞれに`if (selected != null)`が出てきて重複する。

```java
public void mouseDown() {
   selected = findFigure();
   if (selected != null) {
      mode = new SingleSelection(selected); // 図形を動かす担当
   } else {
      mode = new MultipleSelection();       // 範囲選択の担当
   }
}

public void mouseMove() {
   mode.mouseMove(); // どっちのモードかは気にしない
}
```

`mouseDown`の時点でその後の挙動が決まる。判断を1回にまとめて、その結果をオブジェクトとして持ち回るイメージ。
TDD的には、最初は`if`で愚直に通して、重複が見えてきたらこの形にリファクタリングする。

### Imposter（なりすまし）

既存のオブジェクトと同じインターフェースで、実装だけ違うオブジェクトを作って新しいケースに対応するパターン。
正体はただのインターフェースの使い方。新しいケースが出てきたら`if`を足すんじゃなくて、なりすましのクラスを足すという手を思い出すための名前。

例：Null Object（何もしないオブジェクトで`null`チェックの`if`を消す）、Composite、多国通貨の`Money`と`Sum`

### Composite

1個のものと、それを複数まとめたものを同じインターフェースで扱えるようにするパターン。Imposterの一種で、まとめたものが1個のものになりすましてる。
ポイントは入れ子にできること（ツリー構造）。

多国通貨の`Sum`がこれ。`Sum`は`Expression`を2つ持っていて、自分も`Expression`だから式の中に式を入れられる。

```java
interface Expression {
   Money reduce(Bank bank, String to);
}

// 1個のもの
class Money implements Expression {
   public Money reduce(Bank bank, String to) {
      int rate = bank.rate(currency, to);
      return new Money(amount / rate, to); // 指定の通貨に換算
   }
}

// まとめたもの
class Sum implements Expression {
   Expression augend; // MoneyでもSumでもOK
   Expression addend;

   public Money reduce(Bank bank, String to) {
      int amount = augend.reduce(bank, to).amount
                 + addend.reduce(bank, to).amount;
      return new Money(amount, to);
   }
}
```

```java
// (5ドル + 10フラン) + 3ドル　※2フラン = 1ドル
Expression sum = new Sum(
   new Sum(Money.dollar(5), Money.franc(10)),
   Money.dollar(3)
);
Money result = sum.reduce(bank, "USD"); // 13ドル
```

`reduce()`を1回呼ぶだけで、中の`Sum`も順に計算してくれる。
