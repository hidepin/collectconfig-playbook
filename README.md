# collectconfig-playbook

## 仕様

- デフォルトでsosreportは取得しない
- sosreport取得時、デフォルトで/var/tmp/sosreport-を削除する。(sosreport_delete=falseで削除しないことも可能)
- 取得ファイル一覧にあるファイルをすべて取得しようとする。存在しない場合は処理をskipしエラーとはならない。
- 取得するファイル名は完全一致のみ取得可能(アスタリスク、ディレクトリ指定は不可)
- 取得ファイル、取得コマンドの結果は、{{ output }}/{{ hostname }}配下に取得元のディレクトリ構成で取得する

## 設定

### 取得ファイル設定

取得対象ファイルを指定するには
group_vars/allの *collect_files* に取得ファイルを指定する

### 取得コマンド設定

コマンド実行結果を取得するには
group_vars/allの *collect_cli_commands* に実行するコマンドと、出力するファイル名を指定する

- filename: 実行結果を保存するファイル名を設定する
- command: 実行するコマンド

## 実行方法

### 通常実行

'''
ansible-playbook site.yml
'''

### sosreport取得実行

'''
ansible-playbook site.yml -e collect_sosreport=true
'''

### outputディレクトリ指定実行

デフォルトcollectdataの取得先ディレクトリをsampleに変更する場合

```
ansible-playbook site.yml -e output=sample
```

### outputディレクトリを指定しsosreportを取得

'''
ansible-playbook site.yml -e collect_sosreport=true -e output=sample
'''

### sosreportを取得する場合、対象ホストで削除を実施しない場合(/var/tmp/sosreport-を削除しない)

'''
ansible-playbook site.yml -e collect_sosreport=true -e sosreport_delete=false
'''
