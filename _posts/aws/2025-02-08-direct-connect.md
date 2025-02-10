---
layout: post
title: "Direct Connectについて調べた"
date: 2025-02-10
categories: aws directconnect
---

- TOC
{:toc}

# Direct Connect について調べた

いろいろとわからないことが多かったので調べた

## Direct Connect とは

- オンプレミス環境と AWS を専用ネットワーク回線で接続したい場合に使用
- ただ、正確にはオンプレと AWS を専用線でつなぐサービスを提供しているわけではない
- 正確には Direct Connect ロケーション（AWS が提供するデータセンター）と AWS を接続する
- 異なるリージョンや、異なるアカウントに対しての接続にはDX-GWを組み合わせる必要がある

![]({{site.baseurl}}/images/aws/direct-connect/dx-vifs.png)

図にある単語の説明

- Direct Connection Location
  - Direct Connection の接続ポイント
  - AWS と物理的に接続されている
- Customer Router
  - 物理的なルーター
  - オンプレと接続
- VLAN
  - ひとつの Direct Connect 接続に対して複数の VLAN の作成が可能
  - VLAN ごとに VIF を割り当てる
- VIF
  - Public Virtual Interface: VPC に接続
  - Private Virtual Interface： AWS のパブリックサービス（S3, DynamoDB 等）に接続
  - Transit Virtual Interface: DX-GW を経由して複数の VPC と接続

## Direct Connect Gateway とは

- 単一のDirect Connect接続を複数のVPCに共有できる
- 複数のAWSリージョンやVPCに
