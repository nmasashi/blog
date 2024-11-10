---
layout: post
title: "生成AI体験ワークショップ"
date: 2024-11-09
categories: aws ai
---

- TOC
{:toc}

# 生成AI体験ワークショップ

下記ハンズオンを実施した記録

https://catalog.workshops.aws/generative-ai-use-cases-jp/ja-JP/overview/architecture

# 1.事前準備

1-b.2. 基盤モデルの有効化を実施しようとしたところ、モデルがすべて使用不可になっており、有効化できない。。。

↓を参考にAWSで問い合わせを行う
https://zenn.dev/sunwood_ai_labs/articles/amazon-bedrock-claude-3-5-sonnet-application-app


問い合わせたところ、rootユーザーのパスワードの変更とMFAデバイスの更新をもとめられたので更新

```
ご担当者様

本アカウントについてお調べしましたところ、セキュリティ上の懸念により一部サービスに制限がある状況となっておりました。

そのため、セキュリティ強化の対応として MFA の再設定ならびにルートユーザーのパスワードの変更をご対応いただく必要がございます。
※MFA は既に設定いただいている状況かと存じますが、設定されたのが数カ月前のようでございましたので、
既存のものを無効化いただきました上で再度ご設定をいただければと存じます
```

更新後、二日程度でモデルがリクエストできるようになった。
![]({{site.baseurl}}/images/aws/generative_ai_use/access_model.png)

# 2.アプリケーションの構築

AWS CDKを使用した手順を選択

## 2-b.1.1. SageMaker Studio Code Editor の作成

cloudformationを利用してSageMaker Studio Code Editorを作成。SageMaker Studio Code Editorとは機械学習用の統合開発環境のようなもの。

10分程度で完成

![]({{site.baseurl}}/images/aws/generative_ai_use/sagemaker_studio.png)

アプリのソースをCode Editorが動いているサーバーにダウンロード & インストール

![]({{site.baseurl}}/images/aws/generative_ai_use/app_source_download.png)

RAG(Retrieval Augmented Generation)の有効化は任意みたいだがやってみる。

RAG: 大規模言語モデルに外部情報の検索を組み合わせて、回答精度を向上させる技術

「Knowledge Bases for Amazon Bedrock を使用した手順」の方を実施

cdk.jsonで設定を変更してコマンドでデプロイ

# 3. RAG 用データの追加

S3にPDFファイルをアップロードして、それを活用できるようにする。
- S3にデータ（PDFファイル）をアップロード
- Knowledge Baseと同期

# 4. アプリケーションの確認

まずはアカウント作成。作成後にcognitを確認するとアカウントが作成されている様子があった。

![]({{site.baseurl}}/images/aws/generative_ai_use/cognit.png)

チャット機能はこんな感じ。chatGPTと似てる。

![]({{site.baseurl}}/images/aws/generative_ai_use/chat.png)

RAGチャット機能を使用するとS3にアップロードした情報をもとに回答してくれる

![]({{site.baseurl}}/images/aws/generative_ai_use/rag_wifi.png)

全然関係ないこと聞くとこんな感じ

![]({{site.baseurl}}/images/aws/generative_ai_use/rag_noresult.png)

学生の時に書いた論文をアップロードしてみた

![]({{site.baseurl}}/images/aws/generative_ai_use/rag_ad.png)

要約機能もあり、ニュースの要約をしてみた

![]({{site.baseurl}}/images/aws/generative_ai_use/summarize.png)

webコンテンツ抽出機能というものあり、URLを指定するとそのページを要約してくれる

![]({{site.baseurl}}/images/aws/generative_ai_use/web_contents.png)

5. [実践] プロンプトエンジニアリング入門

- LLM: Wikipediaなどの大量テキストデータを使用して事前学習して作成される
- 基盤モデル：事前学習によって作成されたモデル
- ラベル付きデータ：質問と回答の組み合わせデータなど

ラベル付きデータを使用して、基盤も出るを事後学習すると、さらにタスクに特化したモデルを作成できる。

学習したLLMがやっていることは入力されたテキストの次に来る確率が高い単語を予測している。
予測データにないことは答えられない（答えさせても事実と異なる内容になる）。回答の妥当性は自身で検証する必要あり。

分類のフォーマットはこんな感じ

```
以下の3つのテキストの内容をpositive、negative、dosukoiのいずれかに分類してください。
分類例は以下の通りです。
<example>
この商品はとても良い: positive
この商品はちょっと重たい: negative
国技館で見る相撲は素晴らしい: dosukoi
このお相撲さんはかなり大きいです: dosukoi
私は相撲が大好きです: dosukoi
</example>
回答は以下の形式で行なってください。
<format>
## 判定結果
- テキスト
  分類
- テキスト
  分類
</format>

この商品はとても使いにくいです。持ち上げようとしたら肩が外れてしまいました。
この力士はとても強くて頼もしいです
この本はとても面白いです。友達にも薦めたいです
```

![]({{site.baseurl}}/images/aws/generative_ai_use/bunrui.png)


こんなのもできる

```
契約書が必要な理由は何ですか。<context>の中に記載されたテキストから当てはまるものを全て回答してください。

以下の<format>で指定されたフォーマットに従って回答してください。フォーマットで指定されていないことは絶対に言わないでください。

<format>
## 回答
n. 理由
理由が複数ある場合はnの数字を1からインクリメントして続けて記載
</format>

<context>
...
</context>

```

![]({{site.baseurl}}/images/aws/generative_ai_use/context.png)


こんな感じでガイドを付けることもできる

入力

```
あなたは文章校正のプロフェッショナルです。<text>タグの中に記載されたテキストが、<guideline>タグの中にマークダウン形式で記載されたガイドラインを注意深く確認し、テキストがガイドラインに沿っているかを確認し、ガイドラインに沿っていない部分を修正した文章を出力してください。ガイドラインに反していない部分は修正しないでください。必ず<format>タグの形式に沿って出力してください。
<text>
日本がW杯で優勝した、うれしくてご飯を三回おかわりしてしまった。この気持ちをウィンドウズPCを使ってtwｉttｅrで表現することにする。明日は別のW杯の試合があるからeメールで友達を２人誘う。
</text>
<format>
## 校正後の文章
ここに校正後の文章を記載する。
## コメント
ここに校正によって修正された全ての箇所とその理由を以下の形式で記載する。修正されていない箇所は記載しない。
修正前の表現->修正後の表現：修正された理由をガイドラインを引用して説明する
</format>
<guideline>
## 以下の文字は、いわゆる半角を用います。
### ラテン文字
T H E Q U I C K B R O W N F ...
t h e q u i c k b r o w n f ...
### アラビア数字
0 1 2 3 4 5 6 7 8 9
## 数字
数字は原則としてアラビア数字を用います。
例： 1月、100円、3,000ドル、50冊、1本、約10年、明治3年、3時30分、3.14、ナポレオン3世
### アルファベットを使うもの
#### 商品名、ブランド名、企業名などの固有名詞
例： Windows、CD-ROM、iPod、IBM、CI戦略
### 略称表記はできるだけ使いませんが、煩雑な場合は初出の時に併記して以下は略称を用いても構いません。
例： W杯 → ワールドカップ、 eメール → 電子メール
</guideline>
```

出力

```
校正後の文章
日本がワールドカップ（W杯）で優勝した、うれしくてご飯を3回おかわりしてしまった。この気持ちをWindowsパソコンを使ってTwitterで表現することにする。明日は別のW杯の試合があるから電子メール（eメール）で友達を2人誘う。

コメント
W杯->ワールドカップ（W杯）：ガイドラインによると、「略称表記はできるだけ使いませんが、煩雑な場合は初出の時に併記して以下は略称を用いても構いません。」とあるため、初出時に正式名称と略称を併記しました。

三回->3回：ガイドラインによると、「数字は原則としてアラビア数字を用います。」とあるため、漢数字をアラビア数字に修正しました。

ウィンドウズPC->Windowsパソコン：ガイドラインによると、「商品名、ブランド名、企業名などの固有名詞」はアルファベットを使うとあるため、「Windows」と修正しました。また、「PC」は日本語の「パソコン」の方が一般的であるため変更しました。

twｉttｅr->Twitter：ガイドラインによると、「商品名、ブランド名、企業名などの固有名詞」はアルファベットを使うとあるため、正式な表記である「Twitter」に修正しました。

eメール->電子メール（eメール）：ガイドラインによると、「略称表記はできるだけ使いませんが、煩雑な場合は初出の時に併記して以下は略称を用いても構いません。」とあるため、初出時に正式名称と略称を併記しました。

２人->2人：ガイドラインによると、「数字は原則としてアラビア数字を用います。」とあるため、全角数字を半角数字に修正しました。
```

# 6. [発展] IP アドレスの制限

cdk.jsonにアクセスルールを記述してデプロイ

![]({{site.baseurl}}/images/aws/generative_ai_use/iprange.png)

# 9. 後片付け

S3のバケット削除とcloudformationのスタック削除で終わる。らくちん

丸一日動かしてこんな感じ（270円くらい）。大部分がopensearchの料金。。。DB系はやはり高い。。。

![]({{site.baseurl}}/images/aws/generative_ai_use/cost.png)
