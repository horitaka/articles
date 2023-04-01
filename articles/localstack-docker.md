---
title: "LocalStackをdocker composeで起動するサンプル"
emoji: "🌥️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [localstack, docker, aws]
published: true
---

LocalStackをdocker composeで起動する方法です。


**バージョン**
- Docker: Docker version 20.10.17, build 100c701
- LocalStackイメージ: 2023/04時点のもの

# LocalStackとは

LocalStackとはAWSのクラウドサービスをローカル環境でエミュレートするためのツールです。
PythonのCLIもしくはDockerコンテナーを使用してローカル環境から使用することができます。

https://localstack.cloud/

> LocalStack is a cloud service emulator that runs in a single container on your laptop or in your CI environment. With LocalStack, you can run your AWS applications or Lambdas entirely on your local machine without connecting to a remote cloud provider!


# LocalStack起動方法

公式のLocalStackイメージが公開されています。
また、公式サイトにてdocker composeのスクリプトが公開されているのでこれを参考に作成します。
https://docs.localstack.cloud/getting-started/installation/#docker-compose

以下のようなdocker composeファイルを作成します。

```yaml:docker-compose.yaml
version: "3.8"

services:
  localstack:
    container_name: localstack
    image: localstack/localstack
    ports:
      - "127.0.0.1:4566:4566"            
      - "127.0.0.1:4510-4559:4510-4559"
    environment:
      - DOCKER_HOST=unix:///var/run/docker.sock
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
```

dockerコンテナを起動します。
```bash
docker compose up
```

以下のステータスチェックのエンドポイントにリクエストをして正常に起動しているか確認します。
```bash
curl -s localhost:4566/_localstack/init | jq .

{
  "completed": {
    "BOOT": true,
    "START": true,
    "READY": true,
    "SHUTDOWN": false
  },
  "scripts": [
    {
      "stage": "READY",
      "name": "init.sh",
      "state": "SUCCESSFUL"
    }
  ]
}
```

https://docs.localstack.cloud/references/init-hooks/#status-endpoint


# AWSリソースの初期化スクリプト

LocalStackは無料プランではデータが永続化されません。
ローカル環境で起動するたびに初期化データを投入するのは手間ですので、初期化用のスクリプトを作成します。

スクリプトは`/etc/localstack/init/ready.d`に配置することで、Dockerコンテナ起動時に自動で実行されます。

https://docs.localstack.cloud/references/init-hooks/


今回のサンプルではS3にバケットを作成してファイルを初期化するようにします。

```sh: init.sh
#!/bin/bash
awslocal s3 mb s3://test-bucket
cd /home/localstack/data/
awslocal s3 cp sample.txt s3://test-bucket/
awslocal s3 ls s3://test-bucket
```

また、docker-compose.yamlを以下のように修正します。

```diff:docker-compose.yaml
version: "3.8"

services:
  localstack:
    container_name: localstack
    image: localstack/localstack
    ports:
      - "127.0.0.1:4566:4566"            # LocalStack Gateway
      - "127.0.0.1:4510-4559:4510-4559"  # external services port range
    environment:
      - DOCKER_HOST=unix:///var/run/docker.sock
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
+     - "./scripts:/etc/localstack/init/ready.d"
+     - "./data:/home/localstack/data"
```

dockerコンテナを再起動するとログからS3が作成されていることが確認できます。
```bash
docker compose up
...
localstack  | 2023-04-01T01:08:59.352  INFO --- [   asgi_gw_0] localstack.request.aws     : AWS s3.CreateBucket => 200
localstack  | make_bucket: test-bucket
localstack  | 2023-04-01T01:08:59.830  INFO --- [   asgi_gw_0] localstack.request.aws     : AWS s3.PutObject => 200
upload: ./sample.txt to s3://test-bucket/sample.txt               e(s) remaining
localstack  | 2023-04-01T01:09:00.306  INFO --- [   asgi_gw_0] localstack.request.aws     : AWS s3.ListObjectsV2 => 200
localstack  | 2023-04-01 01:08:59         11 sample.txt
```

念のためブラウザからも確認します。
以下にアクセスするとsample.txtの内容が閲覧できます。

http://localhost:4566/test-bucket/sample.txt


LocalStackを利用することで開発時におけるAWS費用を削減でき、効率的に開発やデバッグが行えるようになります。