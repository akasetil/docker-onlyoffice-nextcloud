## Document Server and Nextcloud Docker installation

[元リポジトリ](https://github.com/ONLYOFFICE/docker-onlyoffice-nextcloud)


## 起動手順

### 初期設定

```bash
$ export MYSQL_ROOT_PASSWORD=hogehoge
$ export MYSQL_PASSWORD=hogehoge
$ export maildomain=hoge.com
$ export smtp_user=hogehoge:hogehoge
$ cat docker-compose-template.yml | envsubst > docker-compose.yml
```

### 起動

```bash
$ docker-compose up -d
```

#### s3を保存場所にする場合

この段階で、configを設定してs3にあげるようにしてください  
adminユーザ作成後に変更した場合、自分の環境ではうまく行きませんでした。

事前にs3バケットを作成し、そのバケットのフルアクセス権限をもつIAMユーザーを作成する(IAMロールだとできなかった)

```bash
$ sudo vi /var/lib/docker/volumes/docker-onlyoffice-nextcloud_app_data/_data/config/config.php
```

```php
  'objectstore' =>
  array (
    'class' => 'OC\\Files\\ObjectStore\\S3',
    'arguments' =>
    array (
      'bucket' => '',
      'autocreate' => true,
      'key'    =>  '...',
      'secret' => '...',
      'region' => 'ap-northeast-1',
      'use_ssl' => false,
      'use_path_style' => false,
    ),
  ),
```



### adminユーザー作成

http://<動作しているサーバ>  
にアクセスすると、初期設定画面が表示されますされます

adminユーザのusername, passwordを設定後

データベース種別をMySQLにし、以下のように設定する  
パスワードは先ほど環境変数で指定したもの、その他は以下の画像と同じで良い

![image](https://user-images.githubusercontent.com/58157624/69805922-15612780-1225-11ea-9de9-e3f0aacd5b49.png)


### documet-server設定

正常にadminログインされたら、以下を実行する

```bash
$ pwd
/home/ec2-user/docker-onlyoffice-nextcloud

$ ./set_configuration.sh
```

正常にインストールされたら、新しいドキュメント作成時にWordなどが選べるようになる

![image](https://user-images.githubusercontent.com/58157624/69806344-fdd66e80-1225-11ea-90a8-29750f0bfc13.png)

選ぶと、onlyofficeで編集できる

![image](https://user-images.githubusercontent.com/58157624/69806380-0e86e480-1226-11ea-901e-cf168446a55d.png)


## その他設定

### ユーザーが各自でsign upできるように

毎回管理者がユーザー登録はめんどい。使いたいユーザーが自分でアカウントを作れるように  
[Registration](https://github.com/pellaeon/registration)というアプリを入れるとこれが実現できる

インストールすると、設定に`追加設定`というメニューが表示される  
ここで、デフォで属するグループと、登録を許可するメールドメインを指定できる

また、sign upにメール認証を行うので、メールサーバへの接続設定が必要  
設定->基本設定->メールサーバで以下のように設定する

![image](https://user-images.githubusercontent.com/58157624/69806768-ff546680-1226-11ea-8aae-0403cc0aceac.png)

メールドメイン, username:passwordは、docker-composeで環境変数で指定したものを設定する


### ユーザーで共有のフォルダが欲しい

[Group folders](https://github.com/nextcloud/groupfolders)でできるよ  
特に設定は不要(グループの準備くらい)  
インストールすると、設定画面に共有フォルダのメニューがでる


### ユーザーのデフォルトファイルを削除

以下のフォルダに置かれるファイルが、ユーザー作成時に配置される  
邪魔なので消しておく

```bash
$ sudo ls /var/lib/docker/volumes/docker-onlyoffice-nextcloud_app_data/_data/core/skeleton
Documents  Nextcloud Manual.pdf  Nextcloud intro.mp4  Nextcloud.png  Photos

$ sudo -i
$ cd /var/lib/docker/volumes/docker-onlyoffice-nextcloud_app_data/_data/core/skeleton
$ rm -rf ./*
```


