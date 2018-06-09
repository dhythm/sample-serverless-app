# Amazon Web Services を使ったサーバレスアプリケーション開発ガイド
## Chapter.5 の写経と自己流にカスタマイズした AWS SAM のテンプレート

### バックエンドの設定
ディレクトリを移動する。
```
$ cd spa-backend
```

AWS SAM のデプロイ用バケットを作成する。
```
$ aws s3 mb s3://<BUCKET_NAME> --region ap-northeast-1
```

デプロイパッケージを作成する。
```
$ aws cloudformation package --template-file template.yaml \
> --output-template-file template-output.yaml \
> --s3-bucket <BUCKET_NAME>
```

デプロイする。
```
$ aws cloudformation deploy --template-file template-output.yaml \
> --stack-name <STACK_NAME> --capabilities CAPABILITY_IAM \
> --region ap-northeast-1
```

P.110 の curl コマンドは誤植。
正しくは下記のコマンドを実行する必要がある。
```
$ curl -X POST -d '{"type":"application/json","size":xxx}' \
> https://xxxxxxx.execute-api.ap-northeast-1.amazonaws.com/Prod/images
```

デプロイが完了したらフロントエンド用のキーを確認する。
```
$ aws cloudformation describe-stacks --stack-name <STACK_NAME> \
> --region ap-northeast-1
```

API Gateway の ID を確認する。
```
$ aws apigateway get-rest-apis --region ap-northeast-1
```

S3 バケットに対してイベント通知の設定を行う。
Circular dependency のエラーが発生するため、template.yaml に直接かけない。
ここで指定する __<BUCKET_NAME>__ は、写真格納用のバケット。(AWS SAM のデプロイ用ではない)
```
$ aws s3api put-bucket-notification-configuration \
> --bucket <BUCKET_NAME> \
> --notification-configuration file://notification.json
```
### フロントエンドの設定
src/config.js に API Gateway の ID 、写真格納用の S3 のバケット名、Cognito のプール ID と Client ID を設定する。
```JavaScript
export default {
  Region: 'ap-northeast-1',
  ApiBaseUrl: 'https://xxxxxxxxxx.execute-api.ap-northeast-1.amazonaws.com/Prod',
  ImageBaseUrl: 'https://s3-ap-northeast-1.amazonaws.com/<BUCKET_NAME>',
  UserPoolId: 'ap-northeast-1_xxxxxxxxx',
  UserPoolClientId: 'xxxxxxxxxxxxxxxxxxxxxxxxxx'
}
```
