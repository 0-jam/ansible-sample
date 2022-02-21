# Ansible template

`<user>` はサーバーのログインユーザー名に置き換える

---

1. [インストール](#インストール)
1. [設定](#設定)
1. [Playbook 実行](#playbook-実行)
    1. [Ruby インストール](#ruby-インストール)
    1. [PostgreSQL 設定](#postgresql-設定)

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
(.venv) % ansible-playbook install_dependency.yml -i hosts -u <user>
(.venv) % ansible-playbook install_ruby.yml -i hosts -u <user>
```

### PostgreSQL 設定

```
(.venv) % ansible-playbook setup_postgres.yml.yml -i hosts -u <user> --ask-vault-pass
Vault password:
```
