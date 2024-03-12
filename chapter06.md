# クリーンアップ

ハンズオンを終えたら使用したリソースを削除します。  
削除スクリプトは用意していないので1つずつ手動で削除しましょう。  


3章で作成したリソース
- RDS
  - Name タグが ssm-handson のサブネットグループ
  - Name タグが ssm-handson のRDS
- RDS用のプライベートサブネットとルートテーブル
  - CIDRが 172.31.64.0/20, 172.31.128.0/20 のサブネット
  - Name タグが private のルートテーブル

```bash
# Name タグが ssm-handson のRDSを削除
for db_instance_identifier in $(aws rds describe-db-instances --query "DBInstances[?TagList && contains(TagList[?Key=='Name'].Value, 'ssm-handson')].DBInstanceIdentifier" --output text); do aws rds delete-db-instance --db-instance-identifier $db_instance_identifier --skip-final-snapshot; done

# Name タグが ssm-handson のサブネットグループを削除
aws rds delete-db-subnet-group --db-subnet-group-name ssm-handson

```

```bash
# RDS用のプライベートサブネット削除
 for subnet_id in $(aws ec2 describe-subnets --query 'Subnets[?CidrBlock==`172.31.64.0/20` || CidrBlock==`172.31.128.0/20`].SubnetId' --output text); do aws ec2 delete-subnet --subnet-id $subnet_id; done


# Name タグが private のルートテーブル削除
for route_table_id in $(aws ec2 describe-route-tables --filters "Name=tag:Name,Values=private" --query 'RouteTables[*].RouteTableId' --output text); do aws ec2 delete-route-table --route-table-id $route_table_id; done


```

1章で作成したリソース

- 踏み台用 EC2 インスタンス
  - Name タグが SSMHandson のもの
- インスタンスプロファイル
  - デフォルト名称 SSMHandsonProfile

```bash
# ハンズオン用 EC2 インスタンス削除
InstanceId=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=SSMHandson" | jq -r .Reservations[].Instances[].InstanceId)
aws ec2 terminate-instances --instance-ids $InstanceId

if [ $? == 0 ] ;then echo "Delete Complete.";else echo "Delete failed. check your operations and run once again.";fi
```


```bash
# インスタンスプロファイルの削除
aws iam remove-role-from-instance-profile --instance-profile-name $ROLE_NAME --role-name $ROLE_NAME
aws iam delete-instance-profile --instance-profile-name $ROLE_NAME
aws iam detach-role-policy --role-name $ROLE_NAME --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
aws iam delete-role-policy --role-name $ROLE_NAME --policy-name Handson_chap2_Policy
aws iam delete-role --role-name $ROLE_NAME
```


```bash
# State Manager 関連付けの削除
AssociationId=$(aws ssm list-associations --association-filter-list key=AssociationName,value=Linux_Patch_Apply | jq -r .Associations[].AssociationId)
aws ssm delete-association --association-id $AssociationId
```

```bash
# CloudWatch Logs ロググループの削除
aws logs delete-log-group --log-group-name SSMHandson
```

```bash
# Automation サービスロールの削除
aws cloudformation delete-stack --stack-name SSMHandson
```

```bash
# Automation ランブックの削除
aws ssm delete-document --name SSMHandson
```
