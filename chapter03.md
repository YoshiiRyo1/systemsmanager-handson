# Session Manager Port Forwarding を使った DB 接続


参考：https://aws.amazon.com/jp/blogs/news/use-port-forwarding-in-aws-systems-manager-session-manager-to-connect-to-remote-hosts-jp/

# RDSの準備

踏み台サーバーからPort Forwardingで接続するRDSを作るために、まずはプライベートサブネットを作成します。

サブネットにアタッチするルートテーブルに使用する変数を設定します
```
EC2_ROUTE_TABLE_TAG_NAME='private'
STRING_EC2_ROUTE_TABLE_TAG="ResourceType=route-table,Tags=[{Key=Name,Value=${EC2_ROUTE_TABLE_TAG_NAME}}]" \
&& echo ${STRING_EC2_ROUTE_TABLE_TAG}
```

使用しているVPCのIDを変数として定義します
```
VPC_ID='<今回使っているVPCのID>'
```

設定した変数を使用してサブネットの作成とルートテーブルの関連付けを行います
```
ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id ${VPC_ID} --tag-specifications ${STRING_EC2_ROUTE_TABLE_TAG} --query "RouteTable.RouteTableId" --output text)
SUBNET_ID_AZ_A=$(aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 172.31.64.0/20  --query "Subnet.SubnetId" --output text --availability-zone ap-northeast-1a)
SUBNET_ID_AZ_C=$(aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 172.31.128.0/20  --query "Subnet.SubnetId" --output text --availability-zone ap-northeast-1c)
aws ec2 associate-route-table --route-table-id ${ROUTE_TABLE_ID} --subnet-id ${SUBNET_ID_AZ_C}
aws ec2 associate-route-table --route-table-id ${ROUTE_TABLE_ID} --subnet-id ${SUBNET_ID_AZ_A}
```

作成したサブネットをRDSで明示的に使用するには、明示的にサブネットグループを作成して置く必要があります
なのでサブネットグループを作成します

```
aws rds create-db-subnet-group \
--db-subnet-group-name ssm-handson \
--db-subnet-group-description ssm-handson \
--subnet-ids ${SUBNET_ID_AZ_A} ${SUBNET_ID_AZ_C}
```

次に、作成したサブネットグループを使用するように指定してRDSを作成します

```
aws rds create-db-instance \
--db-name ssmhandson01 \
--db-instance-identifier ssm-handson-01 \
--db-instance-class db.t2.micro \
--engine mysql \
--allocated-storage 20 \
--master-username mymasteruser \
--master-user-password <任意のパスワード> \
--db-subnet-group-name ssm-handson \
--no-multi-az \
--no-publicly-accessible \
--tags Key=Name,Value=ssm-handson
```

マネジメントコンソールでRDSの画面を開き、作成が完了するのを待ちます

# Port ForwardingでRDSに接続する

Port Forwardingを使う前に、一度構成を確認してみましょう

Chapter01で作成した踏み台サーバーが、以下の図の「SSM Managed Instance」に該当します

![alt text](./img/chapter03_portForwarding_architecture.jpg)

（引用：https://aws.amazon.com/jp/blogs/news/use-port-forwarding-in-aws-systems-manager-session-manager-to-connect-to-remote-hosts-jp/ ）

また、今回はCloudShellを使用しているため、「Client workstation」がCloudShellに該当します

構成を確認したところで、実際にPort ForwardingでRDSに接続していきます

まずは、CloudShellと踏み台サーバーの間でセッションを貼ります

```
aws ssm start-session --target <踏み台サーバーのインスタンスID> --document-name AWS-StartPortForwardingSessionToRemoteHost --parameters '{"portNumber":["3306"],"localPortNumber":["1053"],"host":["<RDSのエンドポイント>"]}'
```

以下のような表示がされて入力待ちの状態になったら、CloudShellと踏み台サーバーの間でセッションが張られてます

```
Starting session with SessionId: xxxxx-xxxxxxxxxxxxxxxxx
Port 1053 opened for sessionId xxxxx-xxxxxxxxxxxxxxxxx.
Waiting for connections...

```

別のターミナル画面からRDSへ接続するので、右上の[アクション]から[新しいタブ]をクリックして新しいターミナル画面を開きます



![alt text](./img/chapter03_open_newtab.jpg)

新しいターミナルが開いたら、以下のコマンドでRDSに接続します
（パスワードを聞かれるので、RDS作成時に設定したパスワードを入力してください）

```
mysql -h 127.0.0.1 --port 1053 -u mymasteruser -p
```

以下のような画面が出てきたら、RDSへの接続が成功しています

```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is xxx
Server version: x.x.x Source distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]>
```


 [前へ](./chapter02.md) &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; [次へ](./chapter04.md) 
