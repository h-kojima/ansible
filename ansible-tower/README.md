<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Ansible Tower](#ansible-tower)
  - [Ansible Towerの概要](#ansible-tower%E3%81%AE%E6%A6%82%E8%A6%81)
  - [Ansible Tower関連用語](#ansible-tower%E9%96%A2%E9%80%A3%E7%94%A8%E8%AA%9E)
  - [Ansible Towerのインストール (ver. 3.0.3の情報)](#ansible-tower%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB-ver-303%E3%81%AE%E6%83%85%E5%A0%B1)
  - [Ansible Towerの使い方 (ver. 3.0.3の情報)](#ansible-tower%E3%81%AE%E4%BD%BF%E3%81%84%E6%96%B9-ver-303%E3%81%AE%E6%83%85%E5%A0%B1)
    - [Ansible Tower CLIのセットアップ](#ansible-tower-cli%E3%81%AE%E3%82%BB%E3%83%83%E3%83%88%E3%82%A2%E3%83%83%E3%83%97)
    - [Playbookの実行例](#playbook%E3%81%AE%E5%AE%9F%E8%A1%8C%E4%BE%8B)
    - [ローカルユーザの作成](#%E3%83%AD%E3%83%BC%E3%82%AB%E3%83%AB%E3%83%A6%E3%83%BC%E3%82%B6%E3%81%AE%E4%BD%9C%E6%88%90)
    - [Inventoryのインポート/Dynamic Inventory](#inventory%E3%81%AE%E3%82%A4%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%88dynamic-inventory)
    - [Jobの並列度の設定](#job%E3%81%AE%E4%B8%A6%E5%88%97%E5%BA%A6%E3%81%AE%E8%A8%AD%E5%AE%9A)
    - [Scan Job](#scan-job)
    - [Notification](#notification)
  - [Official Documents](#official-documents)
  - [Revision History](#revision-history)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


# Ansible Tower

## Ansible Towerの概要
Ansible Towerは[Ansible](http://docs.ansible.com/ansible/index.html)による各種自動処理を、管理・追跡するための企業向けソフトウェアです。権限管理、ジョブスケジューラ、ジョブの実行記録といった機能を備えています。Ansible Towerは主に次のような状況で威力を発揮します。  
  
・Ansibleによる自動化は導入したものの、Playbook/Inventoryファイルが乱立しているため、管理コストが増えている。  
どの部署が何のPlaybook/Inventoryを管理しているかを、改めて整理して可視化したい。  
・Ansibleによる実行処理(誰がいつ何を実行して結果はどうだったか)を可視化して、ログとして残したい。  
その際に自分の部署に関係ないものを見たくない、または、関係あるものを見せたくない。  
・Ansibleの知識をあまり知らなくても、開発者が用意したPlaybookを誰でもGUI/CLI/REST APIから簡単に実行できるようにしたい。  
・各部署が開発したPlaybookを連携して実行させるために、API経由で適切な権限を持ったユーザが簡単に実行できるようにしたい。  
・管理対象のホスト情報を可視化して、Playbookによる変更履歴を追跡したい。  
  
Ansible Towerの基盤にはAnsibleを利用していますが、Ansibleの代替品ではありませんので、  
現段階では、汎用性を考えてAnsibleだけでできること([条件分岐](http://docs.ansible.com/ansible/playbooks_conditionals.html)や[GitHubのサービスフックの利用](http://docs.ansible.com/ansible/github_hooks_module.html)など)  
は可能な限りAnsibleだけで実行する方がいいでしょう。  
  
なお、現状のAnsible Towerでは名前空間を考慮していないので、マルチテナントには対応していません。  
例えば、会社/部署毎に重複する名前のリソース(Inventoryなど)をAnsible Towerで作成する  
といったことはできませんので、ご注意ください。

## Ansible Tower関連用語

・Inventory  
Playbookの実行対象となるホスト郡です。Ansible Towerでは、Inventory単位でPlaybookの実行対象を指定します。  

・Project  
Ansible Towerで実行するPlaybookを管理します。Playbookの格納先は、Manual(Ansible Towerのローカルディレクトリ)/Git/Subversion/Mercurialを指定出来ます。Ansible TowerのJobとしてPlaybookを実行するためには、Project/Credential/Inventoryを紐付けたJob Templateを作成する必要があります。  

・Credential  
Playbookを実行するために必要な認証情報です。SSH鍵やクラウドプロバイダの認証情報を登録できます。  
ただし、複数の認証情報を1つのCredentialとして登録することはできません。  

・Job  
Job Templateを利用したPlaybookの実行や実行履歴を管理します。  
また、JobのスケジューリングもWebブラウザで設定できます。

・Organization  
上記項目の管理単位となる項目です。  
Ansible Towerのユーザは割り当てられたOrganizationの中で、上記項目を設定していきます。

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

Ansible TowerはGUIで色々な設定ができますが、管理作業の効率化を考えて基本的にはCLI([`tower-cli`](https://github.com/ansible/tower-cli))の利用を推奨します。

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

Step1. 部署/ホスト/認証情報に相当する、Organization/Inventory/Credentialを作成します。  
管理対象のホストにはSSH公開鍵を配布し、ペアになるSSH秘密鍵をCredentialとして登録します。

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
Playbookは、Ansible Towerの`/var/lib/awx/projects/`以下に保存する必要があります。この保存先のディレクトリは、`/etc/tower/setting.py`で変更できます。変更後は、`ansible-tower-service`コマンドでAnsible Towerのサービスを再起動します。

```
# sed -ie "s/PROJECTS_ROOT\ =\ '\/var\/lib\/awx\/projects'/PROJECTS_ROOT\ =\ '\/var\/lib\/awx\/new-projects'/g" /etc/tower/settings.py
# ansible-tower-service restart
```

また、Projectとして登録するPlaybookは、GitHubのものも指定できます。指定したGitHubからPlaybookをダウンロード(これをAnsible TowerではProjectのUpdateと定義しています)することで、Playbookを実行できるようになります。なお、Job実行時に自動でProjectをUpdateすることもできます。こうしたUpdateに関するオプションは[こちら](http://docs.ansible.com/ansible-tower/latest/html/userguide/projects.html#manage-playbooks-using-source-control)をご参照ください。
```
# tower-cli project create --name Project02 --organization Org01 \
    --scm-type git --scm-url https://github.com/ansible/tower-example.git --scm-update-on-launch true
# tower-cli project update --name Project02
```

Step3. Job Templateを作成してJobを実行します。Jobの実行をスケジューリングしたい場合は、cronなどを利用します。
GUIからJobのスケジューリング設定をしたい場合は、[こちら](http://docs.ansible.com/ansible-tower/latest/html/userguide/job_templates.html#scheduling)をご参照ください。
```
# tower-cli job_template create --name job-user-create01 --job-type run --inventory Inv01 --project Project01 --playbook user-create.yaml --machine-credential Cred01
# tower-cli job launch --job-template job-user-create01
```
JobはWebブラウザから実行することもできます。  
その場合は、Job Templateの横にあるロケットアイコンをクリックします。  
実行結果の情報はCLI/GUIで確認できます。  
実行結果は、`/var/lib/awx/job_status/`に$JOB_ID-$UUID.outという名前で保存されます。  
Webブラウザからダウンロードすることもできます。

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
  
また、システム監査用のユーザ(view権限のみを持つユーザ)も作成できます。

```
# tower-cli user create --username $AUDITOR_USER --password $PASSWORD --email $AUDITOR_USER_MAIL_ADDRESS
# tower-cli user modify --username $AUDITOR_USER --is-system-auditor true
```
  
このユーザは、Ansible Towerで持っている全リソース情報(Organization/Project/Jobなど)を確認できます。  
リソースの作成・編集やJobの実行はできません。
  
なお、ユーザ認証にはローカルで作成する他に、OAuth2/SAML/RADIUS/LDAPも利用できます。  
詳細は[こちら](http://docs.ansible.com/ansible-tower/latest/html/administration/social_auth.html)をご参照ください。

### Inventoryのインポート/Dynamic Inventory

既存のInventoryファイルのインポートやAnsibleの[Dynamic Inventory](http://docs.ansible.com/ansible/intro_dynamic_inventory.html)が利用できますので、Inventoryへのホスト情報の登録コストの削減ができます。 
  
・Inventoryのインポート

`tower-cli`ではなく、Ansible Tower付属の`tower-manage`コマンドを利用します。

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
Scan Jobの設定はGUIで行う必要があります。管理対象ホストの要件やScan Jobの設定方法は[こちら](http://docs.ansible.com/ansible-tower/latest/html/userguide/job_templates.html#scan-job-templates)をご参照ください。  
Scan Jobを実行して、GUIの「Inventory」からホストを選択して、「SYSTEM TRACKING」を選択すると次のような画面を確認できます。

<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/scanjob-01.png" width="100%" height="100%">

ここでScan実行時の差分情報を確認できます。

<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/scanjob-02.png" width="100%" height="100%">
    
<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/scanjob-03.png" width="100%" height="100%">
    
<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/scanjob-04.png" width="100%" height="100%">

### Notification

Ansible Towerでは、ProjectのUpdate/Jobの実行に関するNotification(通知)を設定できます。簡単にテストしてみたい時は、Ansible TowerにPostfixをインストールして、メール通知のテストをすることができます。

```
# yum -y install postfix
# systemctl start postfix; systemctl enable postfix
```

メール通知の設定画面例は以下のようになります。tower-cliでも設定できますが、通知設定の数があまり多くないときは直感的に操作できるGUIの方がいいでしょう。設定が完了したら、鐘アイコンをクリックしてテストを実施できます。

<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/notification01.png" width="100%" height="100%">

設定した通知をJob Templateに紐付けると、Job成功時と失敗時に通知が行われることを確認できます。  
  
・Job Templateで、Job成功/失敗時の通知をONにする

<img src="https://github.com/h-kojima/ansible/blob/master/ansible-tower/images/notification02.png" width="100%" height="100%">

・Job成功時に自動送信されるメッセージ
```
From noreply@noreply.com  Sun Dec 11 15:13:00 2016
Return-Path: <noreply@noreply.com>
X-Original-To: root@localhost
Delivered-To: root@localhost.localdomain
Content-Type: text/plain; charset="utf-8"
Subject: Job #14 'job-scan-01' succeeded on Ansible Tower:
 https://ansible-tower.example.com/#/jobs/14
From: noreply@noreply.com
To: root@localhost.localdomain
Date: Sun, 11 Dec 2016 06:13:00 -0000
Status: RO

Job #14 had status successful on Ansible Tower, view details at https://ansible-
tower.example.com/#/jobs/14

{
    "status": "successful", 
    "credential": "Cred01", 
    "name": "job-scan-01", 
    "started": "2016-12-11T06:12:47.686686+00:00", 
    "extra_vars": "{}", 
    "traceback": "", 
    "friendly_name": "Job", 
    "created_by": "admin", 
    "project": null, 
    "url": "https://ansible-tower.example.com/#/jobs/14", 
    "finished": "2016-12-11T06:13:00.413048+00:00", 
    "hosts": {
        "test2.example.com": {
            "skipped": 1, 
            "ok": 3, 
            "changed": 0, 
            "dark": 0, 
            "failed": false, 
            "processed": 1, 
            "failures": 0
        }, 
        "test1.example.com": {
            "skipped": 1, 
            "ok": 3, 
            "changed": 0, 
            "dark": 0, 
            "failed": false, 
            "processed": 1, 
            "failures": 0
        }
    }, 
    "playbook": "Default", 
    "limit": "", 
    "id": 14, 
    "inventory": "Inv01"
}
```
・Job失敗時に自動送信されるメッセージ
```
From noreply@noreply.com  Sun Dec 11 15:09:45 2016
Return-Path: <noreply@noreply.com>
X-Original-To: root@localhost
Delivered-To: root@localhost.localdomain
Content-Type: text/plain; charset="utf-8"
Subject: Job #13 'job-user-create01' failed on Ansible Tower:
 https://ansible-tower.example.com/#/jobs/13
From: noreply@noreply.com
To: root@localhost.localdomain
Date: Sun, 11 Dec 2016 06:09:45 -0000
Status: RO

Job #13 had status failed on Ansible Tower, view details at https://ansible-towe
r.example.com/#/jobs/13

{
    "status": "failed", 
    "credential": "Cred01", 
    "name": "job-user-create01", 
    "started": "2016-12-11T06:09:33.805764+00:00", 
    "extra_vars": "{}", 
    "traceback": "", 
    "friendly_name": "Job", 
    "created_by": "admin", 
    "project": "Project01", 
    "url": "https://ansible-tower.example.com/#/jobs/13", 
    "finished": "2016-12-11T06:09:45.293444+00:00", 
    "hosts": {
        "test2.example.com": {
            "skipped": 0, 
            "ok": 0, 
            "changed": 0, 
            "dark": 1, 
            "failed": true, 
            "processed": 1, 
            "failures": 0
        }, 
        "test1.example.com": {
            "skipped": 0, 
            "ok": 2, 
            "changed": 0, 
            "dark": 0, 
            "failed": false, 
            "processed": 1, 
            "failures": 0
        }
    }, 
    "playbook": "user-create.yaml", 
    "limit": "", 
    "id": 13, 
    "inventory": "Inv01"
}
```
通知にはメールの他にも、Slack/Twilio/PagerDuty/HipChat/Webhook/IRCを利用できます。設定方法の詳細は[こちら](http://docs.ansible.com/ansible-tower/latest/html/userguide/notifications.html)をご参照ください。
## Official Documents

英語の公式ドキュメントが[こちら](http://docs.ansible.com/ansible-tower/index.html)になります。日本語化はされていません。

## Revision History

2016-12-07 初版リリース
