---
layout: post
title: "Azure OpenAI Serviceで始めるChatGPT/LLMシステム構築入門"
date: 2025-12-29
categories: reading python RAG LangChain LangGraph
---

![image]({{site.baseurl}}/images/reading/azure_openai_dev/bookcover.jpg)

気持ち古めの本だけど勉強用に呼んだのでメモ

## 第 1 章 生成 AI と ChatGPT

人間に好ましい文章を生成することをアライメントと呼ぶ

## 第 4 章 RAG の概要と設計

### 4.4 Azure AI Search

データをインデックス化してそこから検索する

#### フィールドの属性

[参照ページ](https://learn.microsoft.com/ja-jp/azure/search/search-what-is-an-index#field-attributes)

| データ型    | 説明                                                       |
| ----------- | ---------------------------------------------------------- |
| key         | インデックス内のドキュメントの一意識別子                   |
| searchable  | フルテキスト検索可能                                       |
| filterable  | フィルタークエリで利用可能                                 |
| sortable    | デフォルトはスコア順だが、フィールドに基づいて並び替え可能 |
| retrievable | 検索結果に含めるかどうか                                   |

#### 主なデータ型

[参照ページ](https://learn.microsoft.com/ja-jp/rest/api/searchservice/Supported-data-types)

| データ型               | 説明                                                               |
| ---------------------- | ------------------------------------------------------------------ |
| Edm.String             | テキストデータ                                                     |
| Edm.Boolean            | true または false                                                  |
| Edm.Int32              | 32 ビット整数値                                                    |
| Edm.Int64              | 64 ビット整数値                                                    |
| Edm.Double             | 倍精度 IEEE 754 浮動小数点値                                       |
| Edm.ComplexType        | JSON などのの構造化階層データ                                      |
| Collection(Edm.Single) | ベクトルフィールドで使用される単精度 IEEE 754 浮動小数点値のリスト |

Ed は Entity Data Model の略みたい

#### 検索手法

- フルテキスト検索

  BM25 を利用してスコアリングを行っている。つまり、意味を理解しているわけではない。

  検索の流れ

  1. クエリパーサーでクエリを解析
  2. アナライザーが検索分を単語に分解
  3. 検索を実行し、検索スコアに基づいて結果をソート

- ベクトル検索

  埋め込みモデルを活用して文章をベクトルに変換して、検索ワードをベクトル化したものと比較して検索を行う。

- ハイブリット検索

  フルテキスト検索とベクトル検索の組み合わせ

  検索の流れ

  1. テキストとベクトルそれぞれで検索
  2. それぞれの検索結果より新たなランク付けを行う

- セマンティック検索

  フルテキスト検索、またはハイブリット検索された結果に対して、独自の AI モデルによって関連する結果に並び替える。
  クエリの意図を理解した並び替えになるため、こちらの方がただのハイブリットより精度が上がるみたい。

## 第 5 章 RAG の実装と評価

### 5.2.2 ローカル開発環境を構築する

```sh
azd up

## 実行時エラー
ERROR: error executing step command 'provision': deployment failed: error deploying infrastructure: validating deployment to subscription:

Validation Error Details:
POST https://management.azure.com/subscriptions/12345678-1234-1234-1234567890/providers/Microsoft.Resources/deployments/aoai-rag-1767333553/validate
--------------------------------------------------------------------------------
RESPONSE 400: 400 Bad Request
ERROR CODE: InvalidTemplateDeployment
--------------------------------------------------------------------------------
{
  "error": {
    "code": "InvalidTemplateDeployment",
    "message": "The template deployment 'aoai-rag-1767333553' is not valid according to the validation procedure. The tracking id is 'cf7a0b4e-d307-4fe3-bd58-347234470497'. See inner errors for details.",
    "details": [
      {
        "code": "ValidationForResourceFailed",
        "message": "Validation failed for a resource. Check 'Error.Details[0]' for more information.",
        "details": [
          {
            "code": "SubscriptionIsOverQuotaForSku",
            "message": "Operation cannot be completed without additional quota. \r\nAdditional details - Location:  \r\nCurrent Limit (Free VMs): 0 \r\nCurrent Usage: 0\r\nAmount required for this deployment (Free VMs): 1 \r\n(Minimum) New Limit that you should request to enable this deployment: 1. \r\nNote that if you experience multiple scaling operations failing (in addition to this one) and need to accommodate the aggregate quota requirements of these operations, you will need to request a higher quota limit than the one currently displayed."
          }
        ]
      }
    ]
  }
}
--------------------------------------------------------------------------------
```

残念ながらエラー

調べてみるとリソースプロバイダーの設定更新が必要みたい

ということで必要なプロバイダーを有効化
![image]({{site.baseurl}}/images/reading/azure_openai_dev/azure_console_resouce_provider.png)
![image]({{site.baseurl}}/images/reading/azure_openai_dev/azure_console_resouce_provider02.png)

再実行してみたところまだ同じエラー...

![image]({{site.baseurl}}/images/reading/azure_openai_dev/azure_console_quota.png)

そもそも 0 件だった。。。サポートに申請して上限アップしてもらう

1 時間程度で対応してもらえた

![image]({{site.baseurl}}/images/reading/azure_openai_dev/azure_console_quota02.png)

なんかいろいろ作ってくれた

![image]({{site.baseurl}}/images/reading/azure_openai_dev/azure_console_resouces.png)

ストレージアカウントに源頼朝などの wikipedia の情報が入っていてそれを ai search に格納してくれているみたい。

実際に検索してみるとこんな感じ

![image]({{site.baseurl}}/images/reading/azure_openai_dev/search01.png)

フルテキスト検索とベクトル検索の比較もしてみる。

検索ワード「源実友の俳句にはどのような特徴があったのでしょうか？」

- 源実友 → 源実朝
- 俳句 → 和歌

まずはフルテキストであえて誤字を入れて検索

![image]({{site.baseurl}}/images/reading/azure_openai_dev/search02.png)

誤字を考慮した検索になっていないことがわかる。次にベクトルを使用した検索。

![image]({{site.baseurl}}/images/reading/azure_openai_dev/search03.png)

誤字を考慮してくれているような検索結果になっている。
