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

<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/dashboard.png" width="100%" height="100%">

## Ansible Towerの使い方 (ver. 3.0.3の情報)

Ansible TowerはGUIで色々な設定ができますが、管理作業の効率化を考えて基本的にはCLI(tower-cli)の利用を推奨します。

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

### Playbookの実行例
この例では、ある部署が管理しているホストに対して、  
ユーザを作成するための簡単なPlaybookを実行することを想定します。

Step1. Organization/Inventory/Credentialを作成します。

```
  # tower-cli organization create --name Org01
  # tower-cli inventory create --name Inv01 --organization Org01
  # tower-cli host create --name $MANAGED_SERVER1 --inventory Inv01
  # ssh-keygen -f /root/.ssh/id_rsa -N ''
  # ssh-copy-id root@$MANAGED_SERVER1
  # tower-cli credential create --name Cred01 --organization Org01 --kind ssh --ssh-key-data /root/.ssh/id_rsa
```

Step2. PlaybookとProjectを作成します。

```
  # mkdir -p /var/lib/awx/projects/sample-project01
  # cat << EOF  > /var/lib/awx/projects/sample-project01/user-create.yaml
  ---
  - hosts: all
    tasks:
      - name: create user
        user: name=james
  EOF
  # tower-cli project create --name Project01 --organization Org01 --scm-type manual --local-path sample-project01
```

また、Projectとして登録するPlaybookは、GitHub/GitLab/Mercurial/Subversion上のものも指定できます。  

```
  # tower-cli project create --name Project02 --organization Org01 --scm-type git --scm-url https://github.com/ansible/tower-example
```

Step3. Job Templateを作成してJobを実行します。Jobの実行をスケジューリングしたい場合は、cronなどを利用します。

```
  # tower-cli job_template create --name job-user-create01 --job-type run --inventory Inv01 --project Project01 --playbook user-create.yaml --machine-credential Cred01
  # tower-cli job launch --job-template job-user-create01
```

実行結果の情報はCLI/GUIで確認できます。  
実行結果は、`/var/lib/awx/job_status/`に$JOB_ID-$UUID.outという名前で保存されます。  
GUIからダウンロードもできます。

```
  # tower-cli job list
  == ============ ======================== ========== ======= 
  id job_template         created            status   elapsed 
  == ============ ======================== ========== ======= 
   3            7 2016-12-06T10:30:48.357Z successful   10.03
  == ============ ======================== ========== ======= 
```
<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/job.png" width="100%" height="100%">

### ローカルユーザの作成
各部署のサーバ管理者に、Ansible Towerで扱う「部署」(Organization)の管理権限を割り当てると、  
サーバ管理者に割り当てられたサーバのみを、Ansible Towerで管理できるようになります。  
  
この場合は、上記手順で作成した「Org01」をサーバ管理者に割り当てます。

```
  # tower-cli user create --username $USER01 --password $PASSWORD --email $USER01_MAIL_ADDRESS
  # tower-cli organization associate_admin --user $USER01 --organization Org01
```

これにより、「$USER01」ユーザでAnsible Towerにログインすると、「Org01」に関する情報のみが表示されていて、  
他のOrganizationに関する情報(Jobの実行結果など)が隠蔽されていることを確認できます。  
「$USER01」ユーザは「Org01」の中で、Credential/Inventory/Projectの作成・編集ができるようになり、  
自身が管理する認証情報/Playbook/ホスト情報の登録・修正ができるようになります。

### Inventoryのインポート/Dynamic Inventory

既存のInventoryファイルのインポートやAnsibleの[Dynamic Inventory](http://docs.ansible.com/ansible/intro_dynamic_inventory.html)が利用できますので、Inventoryへのホスト情報の登録コストの削減ができます。 
  
・Inventoryのインポート

```
  # tower-manage inventory_import --inventory-name=Inv02 --source=$INVENTORY_FILE_OR_DIRECTORY
```  
・Dynamic Inventory (AWSの例)

Ansible Towerの既存のInventoryに「ec2-Group01」という名前のグループを追加し、  
AWS上で管理しているホスト情報の同期を取るようにします。

```
# tower-cli credential create --name ec2-Cred01 --kind aws --username $AWS_ACCESS_KEY --password $AWS_SECRET_KEY
# tower-cli group create --name ec2-Group01 --source ec2 --credential ec2-Cred01 --inventory $INVENTORY_NAME --update-on-launch true --overwrite true 
# tower-cli group sync ec2-Group01
```

### Jobの並列度の設定

Ansible TowerのエンジンはAnsibleなので、[Ansibleと同じく`ansible.cfg`の`forks`で設定](http://docs.ansible.com/ansible-tower/latest/html/userguide/jobs.html#job-concurrency)します。  
デフォルトの値は5です。

### Scan Job

Ansible Towerに組み込まれたPlaybook`/var/lib/awx/venv/tower/lib/python2.7/site-packages/awx/playbooks/scan_facts.yml`を利用して、  
管理対象のホストの情報(パッケージ/サービス/HW情報)を取得してWebブラウザで確認できます。  
Scan Jobも上記と同様にJob Templateを作成して、Jobを実行します。  
Scan Jobの設定はGUIで行う必要があります。管理対象のシステム要件や設定方法は[こちら](http://docs.ansible.com/ansible-tower/latest/html/userguide/job_templates.html#scan-job-templates)をご参照ください。  
Scan Jobを実行して、GUIの「Inventory」からホストを選択して、「SYSTEM TRACKING」を選択すると次のような画面を確認できます。

<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/scanjob-01.png" width="100%" height="100%">

ここでScan実行時の差分情報を確認できます。

<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/scanjob-02.png" width="100%" height="100%">
    
<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/scanjob-03.png" width="100%" height="100%">
    
<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/scanjob-04.png" width="100%" height="100%">
