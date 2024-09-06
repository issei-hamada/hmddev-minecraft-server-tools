# hmddev-minecraft-server-tools

A tools to create and manage minecraft server on AWS.

## 概要

- Minecraft サーバを AWS 上に構築し、運用する為のテンプレート
- サーバに加えて、運用に必要なリソースも一緒に構築する

## 環境

- OS: Ubuntu 24.04
- Minecraft: 1.21.1
  - バージョンを変えたい場合、template.yaml 内の Instance セクション、Userdata に記述されている wgt URL を最新版に書き換える事

## デプロイ方法

- AWS CloudFormation コンソールより、本テンプレートを選択してデプロイする
- 基本的に、Minecraft 1.21.1 をプレイする分にはパラメータ設定不要
- Outputs タブに表示される IP アドレスを使えば、サーバ接続可能

## 構築リソース一覧

![resources](/images/application-composer-template.yaml.png)

## 機能

### Minecraft サーバ

- 構築完了時点で、Minecraft が起動している
- ポート開放済みかつ固定 Ip も付与される為、友人に連携すればすぐにマルチプレイ可能
- デフォルト設定では、Minecraft で利用する最大メモリサイズは 1024 になっている
  - t3.small の場合の適正サイズ
  - インスタンスサイズによって適宜修正する事

### EC2 Instance Connect Endpoint を使った SSH 接続

- 本テンプレートでは、EC2 Instance Connect Endpoint を構築している
- EC2 コンソールから Minecraft サーバを選んで、「接続」を押下する事で SSH 可能

### DLM を使った スナップショットの自動取得

- Data Lifecycle Manager を使ってサーバのスナップショットを自動で取得する
- 事前/事後スクリプト用の SSM Document も作成する為、Minecraft サービスを停止してからスナップショットを取得する
- デフォルトでは1日１回、毎朝 9:00 に取得して14世代分保管する

### インスタンスの起動・停止スケジューリング

- コスト削減の為、指定した時間で Minecraft サーバを起動・停止する
- デフォルトでは毎日 18:00 に起動し、 3:00に停止する

### Minecraft 運用で必要な監視設定

- Minecraft サーバの稼働状況を監視する為、CloudWatch Agent をインストールし、ログとメトリクスを取得する
- 監視設定について、プロセスのメトリクスに対してアラームを設定する
- Metics Filter で「joinded the game」「left the game」をそれぞれカウントする
  - ダッシュボードで差分を計算すると、現在ログインしているユーザ数を確認できる

### プロセス監視の静観設定

- インスタンス起動・停止をスケジューリングする場合、停止する度にアラームが発報される
- その為、インスタンス起動・停止の直前にアラームをオフにする事が可能
- デフォルトでは毎日 18:15 にアラームが有効になり、2:45 に無効になる
