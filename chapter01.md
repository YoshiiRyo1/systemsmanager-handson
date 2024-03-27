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


## ハンズオン用 EC2 インスタンス（踏み台）の作成

本日のハンズオンは OpsJAWS 運営が用意した EC2 インスタンスを使用します。  

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



