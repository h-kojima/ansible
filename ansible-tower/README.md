# Ansible Tower

## Ansible Towerの概要
Ansible Tower

## Ansible Towerのインストール
Step1. 最新版のRHEL7またはCentOS7サーバを1台(物理でも仮想でも可)用意します。  
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

Step4. ライセンス入力画面が表示されますので、ライセンス(評価版を利用する場合は[こちら](https://www.ansible.com/license)から入手可能)情報を入力すると、Ansible Towerのダッシュボードが表示されます。

<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/dashboard.png" width="50%" height="50%">

## Ansible Towerの使い方
