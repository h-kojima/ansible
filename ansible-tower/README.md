<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Ansible Tower](#ansible-tower)
  - [Ansible Towerの概要](#ansible-tower%E3%81%AE%E6%A6%82%E8%A6%81)
  - [Ansible Towerのアーキテクチャ](#ansible-tower%E3%81%AE%E3%82%A2%E3%83%BC%E3%82%AD%E3%83%86%E3%82%AF%E3%83%81%E3%83%A3)
  - [Ansible Towerのインストール (ver. 3.0.3の情報)](#ansible-tower%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB-ver-303%E3%81%AE%E6%83%85%E5%A0%B1)
  - [Ansible Towerの使い方 (ver. 3.0.3の情報)](#ansible-tower%E3%81%AE%E4%BD%BF%E3%81%84%E6%96%B9-ver-303%E3%81%AE%E6%83%85%E5%A0%B1)
    - [Ansible Tower CLIのセットアップ](#ansible-tower-cli%E3%81%AE%E3%82%BB%E3%83%83%E3%83%88%E3%82%A2%E3%83%83%E3%83%97)
    - [Ansible Towerの](#ansible-tower%E3%81%AE)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


# Ansible Tower

## Ansible Towerの概要
Ansible Tower

## Ansible Towerのアーキテクチャ


## Ansible Towerのインストール (ver. 3.0.3の情報)
Step1. 最新版のRHEL7またはCentOS7サーバを1台(物理でも仮想でも可)用意します。  
システム要件は[こちら](http://docs.ansible.com/ansible-tower/latest/html/installandreference/requirements_refguide.html)をご参照ください。
RHEL7の場合は、Baseチャネルの他にもExtraチャネルの利用が必要です。

```
  ### RHEL7サーバを利用する時のみ実行 ###
  # subscription-manager register
  # subscription-manager repos --disable="*"
  # subscription-manager repos --enable=rhel-7-server-rpms --enable=rhel-7-server-extras-rpms
```

Step2. [こちら](https://releases.ansible.com/ansible-tower/setup-bundle/ansible-tower-setup-bundle-latest.el7.tar.gz)からAnsible Towerインストール用のソフトウェアをダウンロードし、inventoryファイルでパスワードを設定した後に、インストールスクリプトを実行します。
```
  # tar xf ansible-tower-setup-bundle-latest.el7.tar.gz
  # sed -ie "s/admin_password=''/admin_password='$PASSWORD'/g" ansible-tower-setup-bundle-$VERSION.el7/inventory
  # sed -ie "s/redis_password=''/redis_password='$PASSWORD'/g" ansible-tower-setup-bundle-$VERSION.el7/inventory
  # sed -ie "s/pg_password=''/pg_password='$PASSWORD'/g" ansible-tower-setup-bundle-$VERSION.el7/inventory
  # ./ansible-tower-setup-bundle-$VERSION.el7/setup.sh
```
Step3. Ansible Towerのadminユーザのパスワードを設定して、`http://ANSIBLE_TOWER_FQDN`にWebブラウザでアクセスし、adminユーザでログインします。

```
  # tower-manage changepassword admin
```

Step4. ライセンス入力画面が表示されますので、ライセンス(評価版を利用する場合は[こちら](https://www.ansible.com/license)から入手可能)を入力すると、Ansible Towerのダッシュボードが表示されます。

<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/dashboard.png" width="50%" height="50%">

## Ansible Towerの使い方 (ver. 3.0.3の情報)

Ansible TowerはGUIで色々な設定ができますが、管理作業の効率化を考えてCLI(tower-cli)の利用を推奨します。

### Ansible Tower CLIのセットアップ

Step1. tower-cliインストールに必要なEPELリポジトリを有効にして、tower-cliをインストールします。

```
  # yum -y install http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
  # yum -y install python-pip
  # pip install ansible-tower-cli
  # tower-cli --version
```
Step2. Ansible Towerサーバ接続に必要な情報を設定します。この設定ファイルは、`~/.tower_cli.cfg`に保存されます。

```
  # tower-cli config host $ANSIBLE_TOWER_FQDN
  # tower-cli config username admin
  # tower-cli config password $PASSWORD
  # tower-cli user list
  == ======== ================= ========== ========= ============ ================= 
  id username       email       first_name last_name is_superuser is_system_auditor 
  == ======== ================= ========== ========= ============ ================= 
   1 admin    admin@example.com                              true             false
  == ======== ================= ========== ========= ============ ================= 
```

### Ansible Towerの
