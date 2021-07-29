# 手順書

## バージョン一覧

| software     | OS      | PHP | Nginx | Mysql | Laravel |
| :----------: | :-----: | :-: | :---: | :---: | :-----: |
| **ver/type** | centOS7 | 7.3 | 1.21  | 5.7   | 6.0     |

## どういう流れで環境構築したか
### LinuxのCentOSのバージョン7をダウンロード
- 下記のコマンドを実行  
  ```
  vagrant box add centos/7
  ```
- 実行後表示された選択肢の中から3のvirtualboxを選択しEnter  
  ```
    1 ) hyperv
    2 ) libvirt
    3 ) virtualbox
    4 ) vmware_desktop

    Enter your choice: 3
    ```
- 下記のように表示されたら完了
  ```
  Successfully added box 'centos/7' (v1902.01) for 'virtualbox'!
  ```

### vagrant用ディレクトリの作成
- ディレクトリを作成する  
  (今回はvagrant_dirという名前でディレクトリを作成)
- 作成したフォルダの中で下記のコマンドでbox名`(centos/7)`を指定し実行することでvagrantの設定ファイルを生成する
  ```
  `vagrant init centos/7`
  ```

### Vagrantfileの編集
- 下記の`#`を外しゲストOS（Vagrant）の80番の通信をホストOS（MacやWindowsなど）の8080へ転送する設定をする  
また、VagrantfileにてプライベートネットワークIPを192.168.33.19に編集
  ```
  変更点１
  #config.vm.network "forwarded_port", guest: 80, host: 8080
  ↓
  config.vm.network "forwarded_port", guest: 80, host: 8080

  変更点２
  #config.vm.network "private_network", ip: "192.168.33.10"
  ↓
  config.vm.network "private_network", ip: "192.168.33.19"
  ```
- ホストOS(Mac or Windows)のvagrant_dirディレクトリ内とゲストOS(Vagrant)の`/vagrant`のディレクトリ内をリアルタイムで同期するための設定をする
  ```
  変更点３
  config.vm.synced_folder "../data", "/vagrant_data"
  ↓
  config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
  ```

### Vagrant プラグインのインストール
- 下記のコマンドを実行し`vagrant-vbguest`というプラグインをインストールする
  ```
  vagrant plugin install vagrant-vbguest
  ```
  `vagrant-vbguest`は初めに追加したBoxの中にインストールされているGuest Additionsというもののバージョンを、VirtualBoxのバージョンに合わせて最新化してくれるプラグイン
- 下記のコマンドを実行し`vagrant-vbguest`のインストールが完了しているかを確認
  ```
  vagrant plugin list
  ```

### Vagrantを使用してゲストOSの起動
- 下記のコマンドを実行しゲストOSの起動
  ```
  vagrant up
  ```


### ゲストOSへのログイン
- 下記のコマンドを実行しログインする
  ```
  vagrant ssh
  ```

  またはコマンド`vagrant ssh-config`の実行結果の情報を基に下記のコマンドを実行してログイン
  ```
  ssh vagrant@127.0.0.1 -i /Users/xxxx/.vagrant/machines/default/virtualbox -p 2222
  ```

  #### `vagrant ssh-config`の実行結果
  ```
  Host default
  HostName 127.0.0.1
  User vagrant
  Port 2222
  ~省略~
  IdentityFile /Users/xxx/.vagrant/machines/default/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
  ```
  実行後下記のような表記になっていればログイン成功
  ```
  [vagrant@localhost ~]$
  ```

### パッケージをインストール
- 下記のコマンドを実行し、gitなどの開発に必要なパッケージを一括でインストールする
  ```
  sudo yum -y groupinstall "development tools"
  ```

### PHPのインストール
- 下記のコマンドを実行してPHPのバージョン7.3をインストールしPHPのバージョンを確認
  ```
  sudo yum -y install epel-release wget
  sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
  sudo rpm -Uvh remi-release-7.rpm
  sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
  php -v
  ```
### composerのインストール
- 下記のコマンドでPHPのパッケージ管理ツールであるcomposerをインストール
  ```
  php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
  php composer-setup.php
  php -r "unlink('composer-setup.php');"
  ```
- どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動を行います
  ```
  sudo mv composer.phar /usr/local/bin/composer
  ```
- composer のバージョンが確認できたらOK
  ```
  $ composer -v
     ______
    / ____/___  ____ ___  ____  ____  ________  _____
   / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
  / /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
  \____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                      /_/
  Composer version 2.1.4 2021-07-22 13:55:24

  ~省略~
  ```

### laravel6.0のインストール
- ホストOS上のvagrant用ディレクトリで下記のコマンドを実行  
  今回は`laraPro`という名前のプロジェクトにした
  ```
  cd vagrant_dir
  composer create-project --prefer-dist laravel/laravel laraPro "6.*"
  ```

### laravel6.0 認証機能の追加
- 下記のコマンドをプロジェクトファイルの中で実行し認証機能を追加
  ```
  cd laraPro
  composer require laravel/ui "^1.0" --dev
  php artisan ui vue --auth
  ```

- 下記のコマンドを実行しvagrantを再起動し作成したlaravelアプリケーションをゲストOS内の`/vagrant`ファイルと同期させ、再度ログイン
  ```
  vagrant reload
  vagrant ssh
  ```

### NginXのインストール
- 下記のコマンドを実行しviエディタを使用して以下のファイルを作成、修正する
  ```
  sudo vi /etc/yum.repos.d/nginx.repo
  ```

  以下の内容を書き込む
  ```
  [nginx]
  name=nginx repo
  baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
  gpgcheck=0
  enabled=1
  ```

- 書き終えたら保存して、以下のコマンドを実行しNginxのインストールを実行する
  ```
  sudo yum install -y nginx
  nginx -v
  ```
  
- 下記のコマンドでNginxの起動する
  ```
  sudo systemctl start nginx
  ```
  ブラウザにて http://192.168.33.19と入力し、NginxのWelcomeページが表示されたら問題なく動いているので次に進む

- NginxのWelcomeページが表示されず、`Forbidden 403`というエラーが出た場合の対処法  
  viエディタを使用してSELinuxの設定を変更
  「SELinux コンテキスト」の不一致によりエラーが出ているので、SELinuxを無効化(今回はローカル環境なので無効にしても問題ないが、本番環境構築のときは別のアプローチが必要)

  ```
  sudo vi /etc/selinux/config
  ```
  viエディタが開き設定ファイルが表示されるので下記の部分を探す
  ```
  # This file controls the state of SELinux on the system.
  # SELINUX= can take one of these three values:
  # enforcing - SELinux security policy is enforced.
  # permissive - SELinux prints warnings instead of enforcing.
  # disabled - No SELinux policy is loaded.
  SELINUX=enforcing
  ```
  こちらの記述を下記のように書き換えて、保存
  ```
  SELINUX=disabled
  ```
  設定を反映させるためにゲストOSを再起動する必要があるので、ゲストOSをから一度ログアウトして下記コマンドを実行
  ```
  exit
  vagrant reload
  ```

  リロードが完了したら再度ゲストOSにログイン
  ```
  vagrant ssh
  ```

  再度Nginxを起動
  ```
  sudo systemctl start nginx
  ```
  NginxのWelcomeページが表示されたら次に進む

- Nginxは、php-fpmとセットで使用するため、php-fpmにも設定ファイルが存在しているのでこちらも編集する  
使用しているOSがCentOSの場合、`/etc/nginx/conf.d`ディレクトリ下の`default.conf`ファイルが設定ファイルとなる
  ```
  sudo vi /etc/nginx/conf.d/default.conf
  ```

  ```
  server {
    listen       80;
    server_name  192.168.33.19; # Vagranfileでコメントを外した箇所のipアドレスを記述
    # 192.168.33.19(ipを変えた方は設定したip)に対してリクエストがあった場合に、サーバ内のどこのディレクトリを参照するか決める指定
    root /vagrant/laraPro/public; # 追記
    index  index.html index.htm index.php; # 追記

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        #root   /usr/share/nginx/html; # コメントアウト
        #index  index.html index.htm;  # コメントアウト
        try_files $uri $uri/ /index.php$is_args$args;  # 追記
    }

    ~省略~

    # 該当箇所のコメントを解除し、必要な箇所には変更を加える
    # 下記は root を除いたlocation { } までのコメントが解除されていることを確認

    location ~ \.php$ {
    #    root           html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
        include        fastcgi_params;
    }

    ~省略~
  ```

- 次に php-fpm の設定ファイルを編集する
  ```
  sudo vi /etc/php-fpm.d/www.conf
  ```
  変更箇所は以下になる
  ```
  ;24行目近辺
  user = apache
  ↓
  user = nginx

  group = apache
  ↓
  group = nginx
  ```
- Nginxを再起動し、php-fpmを起動する
  ```
  sudo systemctl restart nginx
  sudo systemctl start php-fpm
  ```

  再度ブラウザにて、 http://192.168.33.19 を入力して確認
  画面は表示されるが、以下のようなLaravelのエラーが表示される
  ```
  The stream or file "/vagrant/laravel_app/storage/logs/laravel.log" could not be opened: failed to open stream: Permission denied
  ```

  これは先程php-fpmの設定ファイルの`user`と`group`を`nginx`に変更したが、ファイルとディレクトリの実行`user`と`group`に`nginx`が許可されていないため起きているエラー

- 試しに以下のコマンドを実行する
  ```
  cd /vagrant/laraPro
  ls -la ./ | grep storage && ls -la storage/ | grep logs && ls -la storage/logs/ | grep laravel.log
  ```

  出力結果から、`storage`ディレクトリも`logs`ディレクトリも`laravel.log`ファイルも全て`user`と`group`が`vagrant`となっているので、これでは`nginx`というユーザーの権限をもって`laravel.log`ファイルへの書き込みができない

  以下のコマンドを実行して`nginx`というユーザーでもログファイルへの書き込みができる権限を付与する
  ```
  sudo chmod -R 777 storage
  ```

  再度ブラウザにて、 http://192.168.33.19 を入力して確認
  ブラウザ上でLaravelのwelcomeページが表示されていればOK

### データベースのインストール
- 下記のコマンドを実行しmysql5.7をインストールし、versionの確認ができたらインストール完了
  ```
  sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
  sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
  sudo yum install -y mysql-community-server
  mysql --version
  ```

- 次にMySQLを起動し接続
  ```
  sudo systemctl start mysqld
  mysql -u root -p
  Enter password:
  ```
  今回はデフォルトでrootにパスワードが設定されてしまっているので、
まずはpasswordを調べ、接続しpassswordの再設定を行っていく
  ```
  sudo cat /var/log/mysqld.log | grep 'temporary password'
  2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge
  ```
  `hogehoge`と記載されている箇所に存在するランダムな文字列がパスワード

- 先程出力したランダム文字列をコピー後、再度以下のコマンドを実行し、パスワード入力時にペースト
  ```
  mysql -u root -p
  Enter password:
  ```

- 次に接続した状態でpasswordを変更
  ```
  mysql > set password = "新たなpassword";
  ```

- 新たなpasswordには、必ず大文字小文字の英数字 + 記号かつ8文字以上の設定をする必要がある  
MySQL5.7のパスワードポリシーは厳格で開発段階では非常に面倒のため、以下の設定を行いシンプルなパスワードに初期設定できるようにMySQLの設定ファイルを変更
  ```
  sudo vi /etc/my.cnf
  ```

  ```
  ~省略~

  [mysqld]

  ~省略~

  # read_rnd_buffer_size = 2M
  datadir=/var/lib/mysql
  socket=/var/lib/mysql/mysql.sock

  # 下記の一行を追加
  validate-password=OFF
  ```

- 編集後はMySQLサーバの再起動が必要
  ```
  sudo systemctl restart mysqld
  ```

### データベースの作成
- Laravelのアプリケーションを動かす上で使用するデータベースの作成を行う
  ```
  mysql > create database laravel_app;
  ```
  Query OKと表示されたら作成は完了

### Laravelを動かす
- laraProディレクトリ下の`.env`ファイルの内容を以下に変更
  ```
  DB_PASSWORD=
  ↓
  DB_PASSWORD=登録したパスワード
  ```

- laraProディレクトリに移動して`php artisan migrate`を実行
マイグレーションが問題なく実行できた後、ブラウザ上でユーザー登録ができればローカルで動かしていたLaravelを仮想環境上で全く同じように動かすことができたということになる


## 環境構築の所感
環境構築をする上で何をしようとしているかなどの全体像の把握や、それをするためにどうすべきか、それを実行する際の扱っているものに対する理解などが重要だと感じました。  
また、手順書作成の中で今後同じような手順で作業をする際、非常に有効だと感じました。

## 参考サイト
### Qiita
- [laravel6.0(laravel/ui)でのLogin機能の実装方法](https://qiita.com/daisu_yamazaki/items/a914a16ca1640334d7a5)
- [Qiita マークダウン記法 一覧表・チートシート](https://qiita.com/kamorits/items/6f342da395ad57468ae3)

### DocBase
- [Markdownの書き方](https://help.docbase.io/posts/13697)
