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

条件分岐（`if`）で振る舞いを切り替えている箇所を、オブジェクトの差し替えで表現するパターン。
同じ条件分岐があちこちに重複して出てきたら使いどころ。

本の例は図形エディタの選択ツール。図形の上でクリックしたかどうかで、マウス操作ごとの処理が変わる。

```java
// 条件分岐がメソッドごとに散らばっている
Figure selected;

public void mouseDown() {
   selected = findFigure();
   if (selected != null) {
      select(selected);
   }
}

public void mouseMove() {
   if (selected != null) {
      move(selected);
   } else {
      moveSelectionRectangle();
   }
}

public void mouseUp() {
   if (selected == null) {
      selectAll();
   }
}
```

`selected != null` の判定が何度も出てくるので、「単一選択モード」と「複数選択モード」をオブジェクトにして差し替える。

```java
SelectionMode mode;

public void mouseDown() {
   selected = findFigure();
   if (selected != null) {
      mode = new SingleSelection(selected);
   } else {
      mode = new MultipleSelection();
   }
}

public void mouseMove() {
   mode.mouseMove();
}

public void mouseUp() {
   mode.mouseUp();
}
```

条件分岐はオブジェクトを選ぶ1か所だけになって、あとはポリモーフィズムに任せられる。
TDD的には、まず`if`で愚直にテストを通して、重複が見えてきたらリファクタリングでこの形にもっていく流れ。
