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
