<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [AWX](#awx)
  - [AWXの概要](#awx%E3%81%AE%E6%A6%82%E8%A6%81)
  - [AWXのインストール](#awx%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)
  - [AWXの使い方](#awx%E3%81%AE%E4%BD%BF%E3%81%84%E6%96%B9)
  - [AWXのAPI](#awx%E3%81%AEapi)
  - [Revision History](#revision-history)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


# AWX

## AWXの概要
[AWX](https://github.com/ansible/awx)はAnsible TowerのOSSコミュニティ版ソフトウェアです。Ansible Towerについての概要は[こちら](https://github.com/h-kojima/ansible/tree/master/ansible-tower#ansible-tower%E3%81%AE%E6%A6%82%E8%A6%81)をご参照ください。AWXは2週間毎のリリースを予定しており、本番環境での利用は想定しておりません。こうした情報をまとめた[FAQ](https://www.ansible.com/awx-project-faq)がありますので、ご参照ください。

## AWXのインストール
Step1. 最新版のRHEL7またはCentOS7サーバを1台(物理でも仮想でも可)用意します。  
RHEL7の場合は、Baseチャネルの他にもExtraチャネルの利用が必要です。

```
### RHEL7サーバを利用する時のみ実行 ###
# subscription-manager register
# subscription-manager repos --disable="*"
# subscription-manager repos --enable=rhel-7-server-rpms --enable=rhel-7-server-extras-rpms
```

[Ansible Engine](https://www.ansible.com/ansible-engine)(Red Hat有償サポート付きのAnsible)を利用する場合は、RHELのExtraチャネルではなくAnsible Engineの専用チャネルを有効化します。

```
### RHEL7でAnsible Engineを利用する時のみ実行 ###
# subscription-manager register
# subscription-manager repos --disable="*"
# subscription-manager repos --enable=rhel-7-server-rpms --enable=rhel-7-server-ansible-2.4-rpms
```

Step2. [AWXはDockerで実行することを前提](https://github.com/ansible/awx/blob/devel/INSTALL.md#prerequisites)としています。そのため、AWXのインストールに必要なソフトウェアを予め導入しておきます。この時EPELリポジトリも有効化します。

```
# yum -y install http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
# yum -y install git docker ansible python-pip
# pip install --upgrade pip
# pip install docker-py
# systemctl start docker.service; systemctl enable docker.service
```

Step3. AWXのGitリポジトリをクローンして、インストールするためのPlaybookを実行します。

```
# git clone https://github.com/ansible/awx/
# cd awx/installer
# ansible-playbook -i inventory install.yml
```

Playbookの実行が完了すると、AWX関連のDockerイメージとコンテナの状態を確認できるようになります。各コンテナのログを参照する場合は、`docker logs`コマンドを実行します。

```
# docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
docker.io/ansible/awx_task   latest              fb3347daf93b        28 hours ago        1.01 GB
docker.io/ansible/awx_web    latest              94c203c89253        28 hours ago        982.8 MB
docker.io/rabbitmq           3                   034ebfbce9a1        7 days ago          124.5 MB
docker.io/memcached          alpine              b07c181167cf        7 days ago          7.677 MB
docker.io/postgres           9.6                 1227c4263c8c        2 weeks ago         265.4 MB
# docker ps
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                                NAMES
5b75e6a14d3d        ansible/awx_task:latest   "/tini -- /bin/sh -c "   25 hours ago        Up 47 minutes       8052/tcp                             awx_task
b5fab20eb4f8        ansible/awx_web:latest    "/tini -- /bin/sh -c "   25 hours ago        Up 47 minutes       0.0.0.0:80->8052/tcp                 awx_web
79e86d4d12ff        memcached:alpine          "docker-entrypoint.sh"   25 hours ago        Up 47 minutes       11211/tcp                            memcached
91561b53f030        rabbitmq:3                "docker-entrypoint.sh"   25 hours ago        Up 47 minutes       4369/tcp, 5671-5672/tcp, 25672/tcp   rabbitmq
cb8d1ac80197        postgres:9.6              "docker-entrypoint.sh"   25 hours ago        Up 47 minutes       5432/tcp                             postgres
# docker logs -f awx_task
```

これらのコンテナは[restart_policyが設定](https://github.com/ansible/awx/pull/228/commits/4236654b0c6e10873dd1ab6c07cff6c7fc4942a6)されているため、管理者が意図的に停止しない限り、AWX関連のコンテナが自動起動されるようになっています。

Step4. http://HOSTNAME_OF_AWX/ にアクセスするとログイン画面が表示されます。80番のポートを使いますので、アクセス出来ない場合は80番のポートが開放されているか確認してください。デフォルトでは管理者用のアカウントが用意されており、admin/password でログインできます。

<img src="https://github.com/h-kojima/ansible/blob/master/awx/images/awx-dashboard.png" width="100%" height="100%">

なお、インストール時に管理者用のアカウント名やパスワードを指定したい場合は、インストール用Playbookの実行前にinventoryファイルを編集しておく必要があります。

```
# cd awx/installer
# echo "default_admin_user=ADMIN_USER_NAME" >> inventory
# echo "default_admin_password=ADMIN_USER_PASSWORD" >> inventory
```

## AWXの使い方
AWXの使い方は[Ansible Towerのユーザガイド](http://docs.ansible.com/ansible-tower/latest/html/userguide/index.html)をご参照ください。

## AWXのAPI
REST APIを使えます。http://HOSTNAME_OF_AWX/api/{v1,v2}/ にアクセスすることで、どんなAPIが用意されているか参照できます。

<img src="https://github.com/h-kojima/ansible/blob/master/awx/images/awx-api-v2.png" width="100%" height="100%">

v1とv2がありますが、現時点での両者の違いは"credential_types"を参照できるか否かとなります。v2でしか"credential_types"を参照できないので、ご注意ください。

## Revision History
2017-10-03 初版リリース
