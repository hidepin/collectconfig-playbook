# collectconfig-playbook

## 仕様

- デフォルトでsosreportは取得しない
- sosreport取得時、デフォルトで/var/tmp/sosreport-を削除する。(sosreport_delete=falseで削除しないことも可能)
- ログイン後rootになる場合、/var/tmp/sosreport-は全ユーザにread権が付与される。
- ログイン後rootになる場合、ファイル収集時ファイルサイズの2倍のメモリが必要になるため最大10MB(デフォルト値)までしか収集しない。
- 取得ファイル一覧にあるファイルをすべて取得しようとする。存在しない場合は処理をskipしエラーとはならない。
- 取得するファイル名は完全一致のみ取得可能(アスタリスク、ディレクトリ指定は不可)
- 取得ファイル、取得コマンドの結果は、{{ output }}/{{ hostname }}配下に取得元のディレクトリ構成で取得する

## 実行環境

- OS:
  - RHEL 7.x
  - CentOS 7.x
- Ansible:
  - 2.4.x
- sshpass

## 初期セットアップ

1. ansibleとsshpassのインストール

  - RHEL7.xの場合
    ```
    yum install -y --enablerepo rhel-7-server-extras-rpms ansible sshpass
    ```

  - CentOS7.xの場合
    ```
    yum install -y ansible sshpass
    ```

## 設定

### 取得ファイル設定

取得対象ファイルを指定するには
group_vars/allの *collect_files* に取得ファイルを指定する

### 取得コマンド設定

コマンド実行結果を取得するには
group_vars/allの *collect_cli_commands* に実行するコマンドと、出力するファイル名を指定する

- filename: 実行結果を保存するファイル名を設定する
- command: 実行するコマンド(full pathで記載すること)

### 取得対象サーバ設定

取得対象サーバを指定するには
productionファイルにIPアドレスを記載する

- 例: 172.16.0.1と172.16.0.2から収集する例

  ```
  172.16.0.1
  172.16.0.2
  ```

### gateway経由の情報収集設定(Optional)

1. group_vars/allにgateway設定を追加する

  - gateway_addressにはgatewayのIPアドレスを設定する。(以下例のxxx.xxx.xxx.xxxを書き換える)
  - gateway_userにはgatewayにアクセスするユーザを設定する。(以下例のrootを書き換える。rootでなくてもよい。)
  - 下記実行例のに従い3行を最終行に追記する

    ```
    echo "gateway_address: 'xxx.xxx.xxx.xxx'" >> group_vars/all
    echo "gateway_user: 'root'" >> group_vars/all
    echo "ansible_ssh_common_args: '-o ProxyCommand=\"sshpass -f {{ gateway_password_file }} ssh -l {{ gateway_user }} {{ gateway_address }} -W %h:%p\"'" >> group_vars/all
    ```

2. gatewayにアクセスするユーザのパスワードファイルを配置する

  - 1の手順のgateway_userのパスワードを配置する

    ```
    echo '<パスワード>' > gatewaypassword
    ```

### 取得対象サーバにログイン後rootで収集するの設定(Optional)

1. ansible.cfgのremote_userをrootからログインユーザに変更する。

    ```
    sed -i 's/root/(ログインユーザ)/' ansible.cfg
    ```

2. group_vars/allにログイン後rootになる設定を追加する。

  - ansible_become_methodにsuを設定する
  - ansible_become_userをrootを設定する
  - become_enabledをtrueに設定する
  - 下記実行例のに従い3行を最終行に追記する

    ```
    echo "ansible_become_method: 'su'" >> group_vars/all
    echo "ansible_become_user: 'root'" >> group_vars/all
    echo "become_enabled: true" >> group_vars/all
    ```

3. ansible_become_passを指定しない場合、コマンド実行時 `-K` オプションを指定して実行する (Optinal)

    ```
    ansible-playbook site.yml -K
    ```

### 取得対象サーバ毎にパスワードが異なる場合の設定(Optional)

1. ansible.cfgのask_passを無効化する

    ```
    sed -i 's/^ask_pass = True/ask_pass = False/' ansible.cfg
    ```

2. productionファイルのIPアドレスの後ろにansible_ssh_passを追記する

- 例: 172.16.0.1と172.16.0.2のパスワードがそれぞれhogeとfooの場合

  ```
  172.16.0.1 ansible_ssh_pass=hoge
  172.16.0.2 ansible_ssh_pass=foo
  ```

3. ログイン後rootになる場合、productionファイルのIPアドレスの後ろにansible_become_passを追記する(Optional)

- 例: 172.16.0.1と172.16.0.2のパスワードがそれぞれhogehogeとfoofooの場合

  ```
  172.16.0.1 ansible_ssh_pass=hoge ansible_become_pass=hogehoge
  172.16.0.2 ansible_ssh_pass=foo ansible_become_pass=foofoo
  ```

## 実行方法

### 通常実行


```
ansible-playbook site.yml
```

### sosreport取得実行

```
ansible-playbook site.yml -e collect_sosreport=true
```

### outputディレクトリ指定実行

デフォルトcollectdataの取得先ディレクトリをsampleに変更する場合

```
ansible-playbook site.yml -e output=sample
```

### outputディレクトリを指定しsosreportを取得

```
ansible-playbook site.yml -e collect_sosreport=true -e output=sample
```

### sosreportを取得する場合、対象ホストで削除を実施しない場合(/var/tmp/sosreport-を削除しない)

```
ansible-playbook site.yml -e collect_sosreport=true -e sosreport_delete=false
```
