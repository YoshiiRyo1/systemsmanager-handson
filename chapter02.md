# Session Manager で EC2 へログイン

Session Manager を使用すると、EC2 インスタンスに SSH や RDP などのポートを開けることなく、暗号化された経路でログインできます。  
AWS CLI またはマネジメントコンソールからリモートセッションを開始できます。  
伝統的な踏み台サーバーを用意する必要はありません。SSH キーの管理といった面倒な作業も不要です。    

Session Manager には以下のような利点があります。  

- IAM を利用したアクセス管理
- 受信ポートを開ける必要がない
- AWS CLI またはマネジメントコンソールからログイン
- EC2 インスタンスとオンプレミスサーバーの両方に対応
- ポートフォワーディングによるリモートアクセス
- Windows、Linux、macOS に対応
- セッションログの記録

## マネジメントコンソールからログイン

マネジメントコンソールから Session Manager を使用して EC2 インスタンスにログインします。  

[EC2](https://us-west-2.console.aws.amazon.com/ec2/home#Instances:) 画面を開きます。  

 [Chapter 1](./chapter01.md) で作成した EC2 インスタンスを使用します。そのインスタンスのチェックボックスを選択して、上部の `接続` ボタンをクリックします。  
![img](./img/chap02_ec2.png)  
  
`セッションマネージャー` タブから `接続` ボタンをクリックします。  

ブラウザの別タブが開き、背景が黒いコンソール画面が表示されます。これで接続完了です。  

試しに簡単なコマンドを実行してみましょう。  

※セッションマネージャーは[Ctrl]+[v]によるペーストができないので、マウス操作でペーストしてください。

```bash
date

uname -n

whoami
```

セッションを終了するには、コンソール画面の右上にある `終了` ボタンをクリックします。  


## AWS CLI からログイン

続いて AWS CLI からログインしてみます。ハンズオン用 EC2 のインスタンス ID をコピーしておきます。  

CloudShell で以下のコマンドを実行します。  

```bash
aws ssm start-session --target インスタンスID
```

プロンプトが `sh-4.2$` に変わり、コマンドを実行できるようになります。  


## セッションデータの記録

Session Manager 経由でログインした際のコマンド実行履歴を記録します。  
S3 と CloudWatch Logs の何れか、または、両方に記録することができます。このハンズオンでは CloudWatch Logs に記録します。  

最初にロググループを作成します。  

```bash
aws logs create-log-group --log-group-name SSMHandson
```

マネジメントコンソールの [Session Manager](https://us-west-2.console.aws.amazon.com/systems-manager/session-manager/preferences) 画面を開きます。  

`編集` をクリックします。  

`CloudWatch logging` の下にある **Enable** にチェックを入れます。  
`Enforce encryption` の下にある **Allow only CloudWatch log groups that are encrypted by customers master keys (CMK)** のチェックを外します。  
`CloudWatch のロググループ` から **SSMHandson** を選択します。  

**保存** をクリックします。  

インスタンスプロファイルに CloudWatch Logs の書き込み権限を追加します。  
CloudShell で以下のコマンドを実行します。  

```bash
cat > chap02_policy.json << EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "logs:DescribeLogGroups",
                "logs:DescribeLogStreams"
            ],
            "Resource": "*"
        }
    ]
}
EOF

aws iam put-role-policy \
    --role-name $ROLE_NAME \
    --policy-name Handson_chap2_Policy \
    --policy-document file://chap02_policy.json
```

権限の追加が終わったら、[マネジメントコンソールからログイン](#マネジメントコンソールからログイン) を再度実行します。  

[CloudWatch Logs](https://us-west-2.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-1#logsV2:log-groups/log-group/SSMHandson) 画面を開きます。  
CloudWatch Logs のロググループ `SSMHandson` にログストリームが作成されています。それを開きます。  

実行したコマンドとその結果が表示されています。  

```json
{
    "eventVersion": "1.0",
    "eventTime": "2024-00-00T00:00:00Z",
    "awsRegion": "ap-northeast-1",
    "target": {
        "id": "i-012345678912"
    },
    "userIdentity": {
        "arn": "arn:aws:sts::nnnnnnnn:assumed-role/xxxx"
    },
    "runAsUser": "ssm-user",
    "sessionId": "xxxx",
    "sessionData": [
        "sh-4.2$ sh-4.2$ date",
        "Thu Jan  4 05:45:59 UTC 2024"
    ]
}
```



 [前へ](./chapter01.md) &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; [次へ](./chapter03.md) 
