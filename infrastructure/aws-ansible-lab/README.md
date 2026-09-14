# AWS Ansible Infrastructure Lab

AWS上にLinux検証環境を構築し、
ネットワーク・SSH・Ansible・Apacheの接続関係を
実際に構築しながら理解するための学習ポートフォリオ。

## Purpose

AWSの各サービスやAnsibleのコマンドを単独で覚えるのではなく、

- ネットワーク
- EC2
- SSH
- Ansible
- Apache
- Git / GitHub

がどのようにつながって動作するかを確認する。

## Environment

- Local: WSL2 / Ubuntu 24.04 LTS
- Cloud: AWS / ap-northeast-1
- EC2 OS: AlmaLinux 9
- Configuration Management: Ansible Core 2.14
- Web Server: Apache httpd
- Version Control: Git / GitHub


## Architecture

基本的な管理経路:

    WSL2 / Ubuntu
         |
         | SSH (TCP/22)
         v
    Ansible Controller (EC2)
         |
         | SSH / Ansible
         | Private network
         v
    web01 (EC2)
         |
         v
    Apache httpd

HTTPの外部動作確認では、WSLからweb01のPublic IPv4へ直接アクセスした。

    WSL
     |
     | HTTP (TCP/80)
     v
    Internet
     |
     v
    AWS Internet Gateway
     |
     v
    web01
     |
     v
    Apache httpd

## AWS Network

検証環境では以下を構築した。

- VPC
- Public Subnet
- Internet Gateway
- Route Table
- Security Group
- EC2

Route Tableでは、

- VPC内部宛通信を `local`
- その他のIPv4通信を Internet Gateway

へルーティングする構成とした。

Security Groupでは役割ごとに通信を制限した。

- Ansible Controller
  - SSH: 検証元のPublic IPからのみ許可
- web01
  - SSH: Ansible ControllerのSecurity Groupからのみ許可
  - HTTP: 検証元のPublic IPからのみ許可

## Ansible

Ansible Controllerからweb01を管理する。

実環境用のinventoryはローカル専用とし、Gitでは管理しない。
公開用として `ansible/inventory.example.ini` を用意する。

Playbook `ansible/apache.yml` では以下を実施する。

1. Apache (`httpd`) のインストール
2. Apacheの起動
3. OS起動時の自動起動を有効化
4. `/var/www/html/index.html` の配置
5. localhostからHTTP 200を確認

## Verification

Ansibleのpingモジュールで、
Controllerからweb01への接続を確認した。

結果:

    web01 | SUCCESS
    changed: false
    ping: pong

Apacheについては、

- web01自身からlocalhostへHTTP 200
- WSLからweb01のPublic IPv4へHTTP 200

の両方を確認した。

## Idempotency

同じPlaybookを再実行し、以下を確認した。

    ok=5
    changed=0
    unreachable=0
    failed=0

既に目的の状態になっている場合、
Ansibleが不要な変更を行わないことを確認した。

## Repository Files

    infrastructure/aws-ansible-lab/
    ├── README.md
    └── ansible/
        ├── apache.yml
        └── inventory.example.ini

実環境用の `ansible/inventory.ini` は `.gitignore` で除外する。

秘密鍵、AWS credentials、ログファイル等もGitでは管理しない。

## Cleanup Policy

AWS上の検証環境は学習終了後に削除する。

削除前に、構成・Playbook・検証結果・トラブルシューティングを
GitHubへ記録する。
