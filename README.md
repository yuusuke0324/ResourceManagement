# プロジェクト リソース管理用サンプル Playbook

RVS のリリースフローで実現していた実機上の資源更新を JAS サーバで実現するためのサンプルを提示。

Ansible を使用して Git レポジトリから取得した資源を、リモートサーバ上の該当ディレクトリ配下の資源にコピーし、  
実行権限、パーミッション、所有者を実機の資源と同期させることを実現。

実機上の資源情報を事前に JSON 形式でリストとして取得、clone した資源とリストを突合し、  
更新があれば資源の差し替え、実行権限等をリストの情報をもとに更新する。

## リソースリストサンプル

JSON 形式のリストは以下のようなサンプルイメージで生成される。  
このリストには資源のパス、実行権限、パーミッション、所有者などの情報が含まれる。

```json
[
  {
    "path": "/script/app/wf/wf.sh",
    "mode": "0755",
    "owner": "user1",
    "group": "group1"
  },
  {
    "path": "/script/app/wf/config.txt",
    "mode": "0644",
    "owner": "user2",
    "group": "group2"
  },
  {
    "path": "/script/lib/utils.py",
    "mode": "0755",
    "owner": "user3",
    "group": "group3"
  }
  // ... and more
]
```

## タスクについて

各ロール内のタスクについて記載

### リソースリストの生成または更新

roles/create_update_resource_list/tasks/main.yml 内で以下のタスク実行：

- ansible.builtin.find: リモートサーバ上の資源を再帰的に検索
- ansible.builtin.fetch: 前回のリストが存在する場合、リモートサーバ上の前回のリストを取得
- ansible.builtin.shell: リモートサーバ上で新しいリストを生成または更新
- ansible.builtin.copy: リモートサーバ上で新しいリストを保存

### リソースの更新

roles/update_resources/tasks/main.yml 内で以下のタスクが実行：

- ansible.builtin.find: リモートサーバ上のクローン先資源を再帰的に検索
- ansible.builtin.file: リモートサーバ上のディレクトリとファイルに対して権限と所有者を設定

## 事前準備

1. inventory 等 playbook 実行にあたり必要な設定はゲスト単位で設定すること

1. 以下の各変数はゲスト毎で変更すること

   - `roles/create_update_resource_list/vars/main.yml`
   - `roles/update_resourcees/vars/main.yml`

## 実行方法

このプロジェクトでは、`ansible-playbook` コマンドを使用して異なるオプション指定での実行例を記載。  
実際の運用イメージとしては、1.リスト生成後、3.対象リソースに絞ったリリースをイメージ。

### 0. リソースリストの生成または更新とリソースの更新を両方実行(default)

このコマンド例は リソースリストの生成または更新と、その後にリソースの更新する。

```bash
ansible-playbook -i your_inventory_file manage_resources.yml
```

### 1. リソースリストの生成または更新のみ実行

このコマンド例は `create_update_resource_list` タグが付与されたタスクだけを実行し、リソースリストを生成または更新する。

```bash
ansible-playbook -i your_inventory_file manage_resources.yml --tags create_update_resource_list
```

### 2. リソースの更新のみ実行

このコマンド例は `update_resources` タグが付与されたタスクだけを実行し、リソースを更新する。

```bash
ansible-playbook -i your_inventory_file manage_resources.yml --tags update_resources
```

### 3. リソースの更新対象を指定して実行

このコマンド例は `update_resources` タグが付与されたタスクだけを実行し、  
さらに -e オプションを使用して `update_target` 変数に更新対象を指定する。
※ `update_target`で指定する Git レポジトリ内に格納資源のパスはゲスト毎に変更すること。

```bash
ansible-playbook -i your_inventory_file manage_resources.yml --tags update_resources -e "update_target=/work/gitrepo/script/app/wf.sh"
```
