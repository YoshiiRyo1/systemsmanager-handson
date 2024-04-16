# Systems Manager ハンズオンセットアップ

それではハンズオンを開始します。  
管理者権限を持った IAM で AWS マネジメントコンソールへログインします。  

右上のメニュー部分にある CloudShell ボタンをクリックして CloudShell を開きます。  
![img](img/cloudshell.png)   

本ハンズオンは CloudShell をローカル PC に見立てて進めます。    
![img](img/handson_diagram.drawio.png)

## Windonw PC を使う場合

Windows PC で本リポジトリをクローンして使う場合はテキストエディタの改行コードを LF に変更してください。  

VSCode の場合は右下にある改行コードをクリックして LF に変更します。  
![img](img/chap01_vscode_lf.png)

## Systems Manager にインスタンスのアクセス許可を設定する

デフォルトでは Systems Manager には EC2 インスタンスへのアクセス権限がありません。  
Systems Manager の各機能を使って EC2 インスタンスを操作するための権限を付与します。  

### インスタンスプロファイルを作成する

以下のコマンドでインスタンスプロファイルを作成し、Systems Manager に必要な最低限のポリシー `AmazonSSMManagedInstanceCore` を付与します。  

ロール名は任意です。以下の変数で好きなロール名に設定して CloudShell 上に貼り付けて実行してください。  

```bash
ROLE_NAME=SSMHandsonProfile
```

変数を定義したら以下のコマンドを CloudShell で実行します。  

```bash
cat > assume-policy.json << EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
EOF

export AWS_PAGER=""

aws iam create-role \
  --role-name $ROLE_NAME \
  --assume-role-policy-document file://assume-policy.json

aws iam attach-role-policy \
  --role-name $ROLE_NAME \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore

aws iam create-instance-profile --instance-profile-name $ROLE_NAME

aws iam add-role-to-instance-profile --role-name $ROLE_NAME --instance-profile-name $ROLE_NAME
```

## ハンズオン用 EC2 インスタンス（踏み台）の作成

ハンズオン用踏み台サーバーを作成します。ハンズオンではデフォルト VPC を使います。  
デフォルト VPC ではなく任意の VPC を使う場合は以下3点を忘れずにお願いします。    

- Internet Gateway がアタッチされているサブネット
- パブリック IP アドレスを付与
- 本ハンズオンを通じたコマンドオプション（デフォルト VPC 前提で資料を作成しています。デフォルト VPC 以外を使う場合はコマンドオプションが追加・変更になることがあります。）


以下のコマンドで EC2 インスタンスを作成します。  

```bash
aws ec2 run-instances \
  --image-id resolve:ssm:/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 \
  --instance-type t3.micro \
  --iam-instance-profile Name=$ROLE_NAME \
  --metadata-options HttpTokens=required \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=SSMHandson}]' 'ResourceType=volume,Tags=[{Key=Name,Value=SSMHandson}]'
```

マネジメントコンソールで EC2 画面を開いてください。  
**SSMHandson** という名前のインスタンスが起動しているはずです。  

![ec2](./img/ec2_bastion.png)


## 解説

ハンズオンを実施するための事前準備を行いました。  

Session Manager 経由で AWS リソースを操作するための踏み台サーバーを作成しました。  
踏み台サーバーには SSM(Systems Manager) を操作するための最小権限を持ったインスタンスプロファイルを割り当てています。これらの権限はインスタンスにインストールされている SSM Agent が正しく動作するために必要です。    


本ハンズオンでは CloudShell を使用します。  
ご自身の PC を使う場合は AWS CLI のインストールと設定、Session Manager プラグインのインストールを行ってください。  

[AWS CLI の最新バージョンを使用してインストールまたは更新を行う](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html#cliv2-linux-install)  
[AWS CLI をセットアップする](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-quickstart.html)  
[AWS CLI 用の Session Manager プラグインをインストールする](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)  


 [前へ](./README.md) &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; [次へ](./chapter02.md) 



