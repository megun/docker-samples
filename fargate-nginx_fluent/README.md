## 概要
AWS Fargateでnginxとfluentコンテナを動かす。
nginxのログはマウントしたボリュームに出力させて、fluentで同じボリュームをマウントしてtailする。
fluentコンテナはawslogsにログ出力する。

### ローカルでdocker動作確認
* 起動
```
docker-compose up --build
```

* http://localhostにアクセスして下記のようにfluentの出力にnginxのaccesslog出力されること確認
```
fluent_1  | 2019-09-03 03:23:46.148419700 +0000 nginx.accesslog: {"message":"172.18.0.1 - - [03/Sep/2019:03:23:45 +0000] \"GET / HTTP/1.1\" 200 134 \"-\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169 Safari/537.36\" \"-\""}
fluent_1  | 2019-09-03 03:23:46.148449700 +0000 nginx.accesslog: {"message":"172.18.0.1 - - [03/Sep/2019:03:23:45 +0000] \"GET /favicon.ico HTTP/1.1\" 404 571 \"http://localhost/\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169 Safari/537.36\" \"-\""}
```

### ECRにイメージプッシュ
nginxとfluentのREADME.md参照


### ECS-CLIインストール
https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/ECS_CLI_installation.html

```
$ sudo curl -o /usr/local/bin/ecs-cli https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-darwin-amd64-latest
$ curl -s https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-darwin-amd64-latest.md5 && md5 -q /usr/local/bin/ecs-cli

$ sudo chmod +x /usr/local/bin/ecs-cli
$ ecs-cli --version
```

### config設定
* configを作成
```
$ ecs-cli configure --cluster tutorial --region ap-northeast-1 --default-launch-type FARGATE --config-name tutorial
INFO[0000] Saved ECS CLI cluster configuration tutorial.

$ cat ~/.ecs/config
version: v1
default: tutorial
clusters:
  tutorial:
    cluster: tutorial
    region: ap-northeast-1
    default_launch_type: FARGATE
```

* credentials設定(~/.ecs/credentials)
```
$ ecs-cli configure profile --access-key AWS_ACCESS_KEY_ID --secret-key AWS_SECRET_ACCESS_KEY --profile-name tutorial
INFO[0000] Saved ECS CLI profile configuration tutorial.

 cat ~/.ecs/credentials
version: v1
default: tutorial
ecs_profiles:
  tutorial:
    aws_access_key_id: AWS_ACCESS_KEY_ID
    aws_secret_access_key: AWS_SECRET_ACCESS_KEY
```

### クラスタ作成&デプロイ
* cluster作成
```
$ ecs-cli up --vpc vpc-xxxxxxxx --subnets subnet-xxxxxxxx --security-group sg-xxxxxxxxxxxxxxxxx
INFO[0001] Created cluster                               cluster=tutorial region=ap-northeast-1
INFO[0032] Waiting for your cluster resources to be created...
INFO[0032] Cloudformation stack status                   stackStatus=CREATE_IN_PROGRESS
Cluster creation succeeded.
```

* タスクデプロイ(cloudwatch-logのグループ作成しておく)
```
$ ecs-cli compose --file docker-compose_ecs.yml --project-name tutorial service up --cluster-config tutorial
```

### 削除
* タスクを停止する
```
$ ecs-cli compose --file docker-compose_ecs.yml --project-name tutorial service down --cluster-config tutorial
INFO[0000] Updated ECS service successfully              desiredCount=0 serviceName=tutorial
INFO[0000] Service status                                desiredCount=0 runningCount=1 serviceName=tutorial
INFO[0015] Service status                                desiredCount=0 runningCount=0 serviceName=tutorial
INFO[0015] (service tutorial) has stopped 1 running tasks: (task f2c062f5-176b-41ac-a4e6-4502561cc461).  timestamp="2018-08-27 15:09:02 +0000 UTC"
INFO[0015] ECS Service has reached a stable state        desiredCount=0 runningCount=0 serviceName=tutorial
INFO[0015] Deleted ECS service                           service=tutorial
INFO[0016] ECS Service has reached a stable state        desiredCount=0 runningCount=0 serviceName=tutorial
```

* クラスタを削除する
```
$ ecs-cli down --force --cluster-config tutorial
INFO[0000] Waiting for your cluster resources to be deleted...
INFO[0001] Cloudformation stack status                   stackStatus=DELETE_IN_PROGRESS
INFO[0031] Deleted cluster                               cluster=tutorial
```
