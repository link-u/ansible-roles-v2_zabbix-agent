# zabbix_agent

## 概要

zabbix-agentのインストール & 監視設定用のロール

## 動作確認バージョン

- Ubuntu 18.04 (bionic)
- ansible >= 2.8
- Jinja2 2.10.3

## 使い方 (ansible)

### Role variables

```yaml
### インストール設定 ###############################################################################
zabbix_install_flag: True
zabbix_install_packages:    # zabbix-agent のインストールに必要なパッケージ
  - python3-pip
  - python3-pymysql
zabbix_repo_package: >-     # zabbix-4.0 の apt リポジトリ追加パッケージ
  https://repo.zabbix.com/zabbix/4.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_4.0-3+{{ ansible_distribution_release }}_all.deb
# ※ zabbix_repo_package について
#   zabbix の公式 apt リポジトリの追加方法は以下のリンク先を参考
#   * 4.0系: https://www.zabbix.com/documentation/4.0/manual/installation/install_from_packages/debian_ubuntu
#   * 5.0系: https://www.zabbix.com/documentation/5.0/manual/installation/install_from_packages/debian_ubuntu


### conf ファイル設定 #############################################################################
## zabbix-agent が zabbix-server にログインするための情報
#  * サンプルなので自環境合わせて要修正
zabbix_agent_kanshi_server_info:
  ip_address: "198.51.100.1"
  url: "http://198.51.100.1:10080"
  login_password: "default&zabbix&password"

zabbix_agent_group_name: "{{ inventory_hostname }}"  # 基本的にプロジェクトやサービスの名前に変更するべき
zabbix_agent_ipv4: "{{ local_ipv4 }}"
zabbix_agent_extra_templates: []

## 設定例
# zabbix_agent_extra_templates:
#   - Nginx
#   - MySQL
#   - PHP-fpm
#   - Postfix
#   - Clickhouse
#   - Rsyncd

## zabbix-agent の設定
zabbix_agent_settings:
  Server: "{{ zabbix_agent_kanshi_server_info.ip_address }}"
  ServerActive: "{{ zabbix_agent_kanshi_server_info.ip_address }}"
  Hostname: "{{ inventory_hostname }}"
  AllowRoot: 1

## mysqlのステータスを取得するのに使うアカウント
# * 自環境に合わせて要修正
zabbix_agent_mysql_root_password: "{{ pxc_root_password | default('default&root&password') }}"
zabbix_agent_mysql_user_name: "zabbix_agent_mysql_user"
zabbix_agent_mysql_password: "zabbix&agent&mysql&password"
zabbix_agent_mysql_conf_file: "/etc/zabbix/my.cnf"
zabbix_agent_mysql_socket: "{{ pxc_socket | default('/var/run/mysqld/mysqld.sock') }}"

## zabbix-agent の conf.d ディレクトリ
# ※ 2系からアップグレードする場合は /etc/zabbix/zabbix_agentd.conf.d を指定
zabbix_agent_conf_dir: "/etc/zabbix/zabbix_agentd.d"

zabbix_hostmacros:
  - hosts: "{{ play_hosts }}"
    name: SLACK_MENTION
    value: "@here"
```

```yaml
- hosts:
    - servers
  roles:
    - { role: zabbix-agent, tags: [ "zabbix-agent" ] }
```

## 後方互換性について

### 削除された変数の一覧


以下の変数は `group_vars` から削除して頂いて大丈夫です.

* `zabbix_repo_root`
  * zabbix の apt リポジトリを追加する方針に切り替えたためこの変数は不要になりました.

* `zabbix_package_names`
  * zabbix-agent を入れることは自明なので直書きするように変更しました.
