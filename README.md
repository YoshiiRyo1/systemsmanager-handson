# AWS Systems Manager Hands-on for Beginners

ようこそ。  
本コンテンツでは Systems Manager のハンズオンを通して AWS 上のシステムを管理する方法を学びます。  

## 対象

- 初めて Systems Manager を使う方
- Systems Manager にまだ慣れていない方
- これから Systems Manager を使いこなそうとする気概のある方

## 目次

1. Systems Manager ハンズオンセットアップ
   1. インスタンスプロファイルの作成
   2. ハンズオン用 EC2 インスタンス（踏み台）の作成
   3. AWS CLI インストール、Session Manager プラグインのインストール
   4. 解説
2. Session Manager で EC2 へログイン
   1. セッションログの設定
   2. ローカル PC からログイン（AWS CLI）
   3. ローカル PC からログイン（Teraterm or Putty）  ※可能であればで
   4. 解説
3. Session Manager Port Forwarding を使った DB 接続
   1. 接続用 RDS クラスターの作成
   2. ローカル PC から DB 接続
   3. 解説
4. State Manager による EC2 管理
   1. State Manager 初期設定
   2. 定期的なジョブを実行する
   3. 解説
5. Automation を使った運用自動化
   1. AWS 管理ランブックを実行してみよう
   2. ランブックを書いてみよう
   3. 解説


## ハンズオンの前提

- 自身が所有している AWS アカウントがあること
  - 本番環境ではないことが望ましい
- AdministratorAccess 権限の持つ IAM ユーザー/ロールが使えること
- インターネットアクセスが可能な VPC があること

## AWS 公式

AWS が公開しているハンズオンにもチャレンジしてみてください。  

[AWS Systems Manager ハンズオン](https://catalog.us-east-1.prod.workshops.aws/workshops/7e60f6e3-0c8f-488a-bedc-632aa8d526ea/ja-JP)  
[AWS Systems Manager 入門ハンズオンを公開しました！– AWS Hands-on for Beginners Update](https://aws.amazon.com/jp/blogs/news/aws-hands-on-for-beginners-20/)  

## 本コンテンツについて

このコンテンツは [OpsJAWS](https://manage.doorkeeper.jp/groups/opsjaws) によって管理されています。  

