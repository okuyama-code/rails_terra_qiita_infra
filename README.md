## クローン後にterraform.tfvarsを作成し、以下の環境変数を自分で設定すること
```
aws_access_key = ""
aws_secret_key = ""
aws_account_id = ""
database_name = "sample_app_production"
database_username = "root"
database_password = "password"
```


## 手順

### ECRリポジトリを作成
Terraformを使用して特定のリソースのみを作成
ecr.tfのやつのみ作成
```
terraform apply -target={aws_ecr_repository.sample_app,aws_ecr_lifecycle_policy.sample_app_lifecycle_policy,aws_ecr_repository.sample_nginx}
```