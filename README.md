# Ansible template

`<user>` はサーバーのログインユーザー名に置き換える

---

1. [インストール](#インストール)
1. [設定](#設定)
1. [Playbook 実行](#playbook-実行)
    1. [Ruby インストール](#ruby-インストール)
    1. [PostgreSQL 設定](#postgresql-設定)
    1. [Capistrano 用設定ファイル配置](#capistrano-用設定ファイル配置)

---

## インストール

venv で Python 3 の環境を作って，そこに Ansible をインストール

```
% python3 -m venv .venv
% source .venv/bin/activate
(.venv) % pip install -U pip
(.venv) % pip install -r requirements.txt
(.venv) % ansible-galaxy collection install community.postgresql
```

## 設定

`hosts` に以下を追記：

```
[appservers]
<VM インスタンスの外部 IP アドレス>
```

インスタンスの SSH 鍵を SSH agent に追加する：

```
% eval $(ssh-agent)
Agent pid 48155
% ssh-add ~/.ssh/gce
Identity added: /Users/jam/.ssh/gce (<user>)
```

## Playbook 実行

### Ruby インストール

```
(.venv) % ansible-playbook install_ruby.yml -i hosts -u <user>
```

### PostgreSQL 設定

`ansible_credentials.yml` を作成して以下のように書く：

```yaml
---
db_password: "database_password"
```

`ansible-vault` コマンドを使って上記のファイルを暗号化：

```
% ansible-vault encrypt ansible_credentials.yml
New Vault password:
Confirm New Vault password:
Encryption successful
```

```
(.venv) % ansible-playbook setup_postgres.yml -i hosts -u <user> --ask-vault-pass
Vault password:
```

`setup_postgres.yml` のデータベース名は必要に応じて書き換える

```yaml
    - name: Create a new database with name "（対象のデータベース名）"
      community.postgresql.postgresql_db:
        name: （対象のデータベース名）
      become: true
      become_user: postgres
    - name: Connect to （対象のデータベース名） database, create database user, and grant access to database and products table
      community.postgresql.postgresql_user:
        db: （対象のデータベース名）
        name: "{{ ansible_facts.env.USER }}"
        password: "{{ db_password }}"
        priv: ALL
      become: true
      become_user: postgres
```

### Capistrano 用設定ファイル配置

Capistrano のデプロイを通すための下準備（Capistrano 自体は別で実行する）

`config/master.key` に Rails プロジェクトの `master.key` をコピーして以下を実行：

```
(.venv) % ansible-playbook deploy_settings.yml -i hosts -u <user>
```

`deploy_settings.yml` のデータベース名は必要に応じて書き換える

```yaml
     - name: Create a directory
       file:
         path: "{{ ansible_facts.env.HOME }}/（Rails プロジェクト名）/shared/config"
         state: directory
         recurse: yes
     - name: Copy master.key
       copy:
         src: "config/master.key"
         dest: "{{ ansible_facts.env.HOME }}/（Rails プロジェクト名）/shared/config/master.key"
```
