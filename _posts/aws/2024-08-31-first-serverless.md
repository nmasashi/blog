---
layout: post
title: "はじめてのサーバーレス"
date: 2023-09-21
categories: aws lambda
---

- TOC
{:toc}

# はじめてのサーバーレス

こちらのハンズオンを実施した記録

https://github.com/harunobukameda/AWS-Amplify-AWS-Lambda-Amazon-DynamoDB-AWS-API-Gateway-Amazon-Cognito/blob/master/Scenario.pdf

Cloud9が使えなくなっているので、頑張って実施してく。。。

# DynamoDB作成

パーティションキーとソートキーを設定してデフォルト設定で作成

![]({{site.baseurl}}/images/aws/first-serverless/dynamo_init.png)

# Lambda用のpolicy作成

![]({{site.baseurl}}/images/aws/first-serverless/lambda_policy.png)
